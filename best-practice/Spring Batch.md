`NotificationModule.java`
```java
@Slf4j  
@RequiredArgsConstructor  
@Component  
public class NotificationModule {  
  
    private final NotificationMapper notificationMapper;  
  
    private final JobLauncher jobLauncher;  
  
    @Qualifier("notificationTriggerJob")  
    private final Job notificationTriggerJob;  
  
    @Qualifier("notificationDispatcherJob")  
    private final Job notificationDispatcherJob;  
  
    private final TimeModule timeModule;  
  
  
    // 1분마다 실행해야 하는 Notification이 없는지 확인한다  
    @Scheduled(cron = "${schedule.cron.reward.publish}")  
    public void notificationTrigger() {  
        log.info("Start notificationTrigger()");  
        LocalDateTime now = timeModule.now();  
        log.info("Test dayOfWeek, hour, minute : {}, {}, {}", now.getDayOfWeek().getValue(), now.getHour(), now.getMinute());  
  
        List<Notification> notifications = notificationMapper.findAllScheduledNotifications(  
            now.getDayOfWeek().getValue(), now.getHour(), now.getMinute());  
  
        log.info("notificationId : {}", notifications.get(0).getId());  
        // notifcations이 비어있지 않다면 현재 실행해야 하는 Notification이 있다고 가정합니다.  
        if (!notifications.isEmpty()) {  
            // notifications을 순회하며 각각의 Notification에 스프링 배치 작업을 진행합니다.  
            notifications.stream()  
                .filter(Notification::isEnabled)  
                .forEach(notification -> {  
                    try {  
                        log.info("Start for-each : {}", notification.getId());  
                        JobParameters jobParameters = new JobParametersBuilder()  
                            .addLong("notificationId", notification.getId())  
                            .toJobParameters();  
  
                        jobLauncher.run(notificationTriggerJob, jobParameters);  
                    } catch (Exception e) {  
                        log.error("Notification Trigger Error", e);  
                    }  
                });  
        }  
    }  
  
    // 1분마다 실행하여 전송해야 하는 Notification이 없는지 확인한다  
    @Scheduled(cron = "0 */1 * * * *")  
    public void notificationDispatcher() {  
        List<Notification> notifications = notificationMapper.findAllUncheckedNotifications();  
  
        if (!notifications.isEmpty()) {  
            notifications.stream()  
                .forEach(notification -> {  
                    try {  
                        JobParameters jobParameters = new JobParametersBuilder()  
                            .addLong("notificationId", notification.getId())  
                            .addLong("staffId", notification.getStaffId())  
                            .addLong("time", System.currentTimeMillis())  
                            .toJobParameters();  
  
                        jobLauncher.run(notificationDispatcherJob, jobParameters);  
                    } catch (Exception e) {  
                        log.error("Notification Dispatcher Error", e);  
                    }  
                });  
        }  
    }  
}
```

`NotificationTrigger.java`
```java
@Slf4j  
@RequiredArgsConstructor  
@Configuration  
public class NotificationTriggerConfig {  
  
    private final DataSource dataSource;  
  
    @Bean  
    public Job notificationTriggerJob(final JobRepository jobRepository,  
        final Step notificationTriggerStep) {  
        log.info("Start notificationTriggerJob()");  
        return new JobBuilder("notificationTriggerJob")  
            .repository(jobRepository)  
            .start(notificationTriggerStep)  
            .build();  
    }  
  
    @Bean  
    @JobScope    public Step notificationTriggerStep(JobRepository jobRepository,  
        PlatformTransactionManager transactionManager,  
        @Value("#{jobParameters['notificationId']}") Long notificationId) {  
        log.info("Start notificationTriggerStep()");  
  
        return new StepBuilder("notificationTriggerStep")  
            // <T1, T2> 에서 T1은 Reader에서 반환하는 타입, T2는 프로세서에서 처리한 후 Writer 에게 전달하는 타입  
            // 추가적으로 chunk size는 page size와 동일하게 가져가는 것이 좋다  
            .<Notification, Notification>chunk(10)  
            .reader(notificationTriggerReader(notificationId))  
            .processor(notificationTriggerProcessor())  
            .writer(notificationTriggerWriter())  
            .repository(jobRepository)  
            .transactionManager(transactionManager)  
            .build();  
    }  
  
    @Bean  
    @StepScope    public JdbcCursorItemReader<Notification> notificationTriggerReader(  
        @Value("#{jobParameters['notificationId']}") Long notificationId) {  
        log.info("Start notificationTriggerReader()");  
  
        return new JdbcCursorItemReaderBuilder<Notification>()  
            .name("notificationTriggerReader")  
            .dataSource(dataSource)  
            .sql(  
                "SELECT id, staff_id, message, target FROM tb_notification_3 WHERE id = ?")  
            .preparedStatementSetter(ps -> ps.setLong(1, notificationId))  
            .rowMapper(new RowMapper<Notification>() {  
                @Override  
                public Notification mapRow(ResultSet rs, int rowNum) throws SQLException {  
                    Notification notification = new Notification();  
                    notification.setId(rs.getLong("id"));  
                    notification.setStaffId(rs.getLong("staff_id"));  
                    notification.setMessage(rs.getString("message"));  
                    notification.setTarget(rs.getString("target"));  
                    return notification;  
                }  
            })  
            .build();  
    }  
  
    @Bean  
    @StepScope    public ItemProcessor<Notification, Notification> notificationTriggerProcessor() {  
        log.info("Start ItemProcessor()");  
  
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);  
        return notification -> {  
            log.info("Start update modDate");  
            jdbcTemplate.update(  
                "UPDATE TB_NOTIFICATION_3 SET MOD_DATE = NOW() WHERE ID = ?",  
                notification.getId());  
            return notification;  
        };  
    }  
  
    // ItemReader에 의해 조회된 Notification 객체들이 Writer로 전달된다.  
    @Bean  
    public ItemWriter<Notification> notificationTriggerWriter() {  
        log.info("Start ItemWriter()");  
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);  
        return notifications -> {  
            for (Notification notification : notifications) {  
                jdbcTemplate.update(  
                    "INSERT INTO tb_notification_4 (notification_id, staff_id, message, target, is_checked) VALUES (?, ?, ?, ?, ?)",  
                    notification.getId(), notification.getStaffId(), notification.getMessage(),  
                    notification.getTarget(), false);  
            }  
        };  
    }  
}
```