간단한 스프링 배치 코드를 테스트 하던 도중 문제가 발생했습니다. 

![[스크린샷 2024-04-29 오후 3.02.26.png]]

당시 제 코드는 다음과 같았습니다.
```java
@RequiredArgsConstructor  
@Component  
public class SimpleJobConfiguration {  
  
    private final JobRepository jobRepository;  
    private final PlatformTransactionManager transactionManager;  
  
    // 여기 주입 되는 Step은 Step의 이름을 토대로 주입되는 것인가?  
    @Bean(name = "jobWithStringParameter")  
    public Job jobWithStringParameter(Step stepWithStringParameter) {  
        return new JobBuilder("jobWithStringParameter", jobRepository)  
            // simpleStep() 이 아닌 DI를 이용해야 합니다.  
            .start(stepWithStringParameter)  
            .build();  
    }  
  
    @Bean(name = "jobWithLongParameter")  
    public Job jobWithLongParameter(Step simpleStep) {  
        return new JobBuilder("jobWithLongParameter", jobRepository)  
            .start(simpleStep)  
            .build();  
    }  
  
    // JobParameter는 @JobScope, @StepScope 실행 시점에 사용되어야 합니다  
    @Bean  
    @JobScope    public Step stepWithStringParameter(@Value("#{jobParameters['name']}") String name) {  
        return new StepBuilder("stepWithStringParameter", jobRepository)  
            .tasklet((contribution, chunkContext) -> {  
                System.out.println("Hello, Spring Batch! My name is" + name);  
                return RepeatStatus.FINISHED;  
            }, transactionManager)  
            .build();  
    }
```

쉽게 말해 JobParameter로 오는 인자에 따라 Job이 구분되어 실행되는지 확인하고 있었습니다. 

문제는 계속해서 Job에 null 값이 설정되는 것이었습니다.
`JobLauncherTestUtils.java`
```java
public JobExecution launchJob(JobParameters jobParameters) throws Exception {  
                return getJobLauncher().run(this.job, jobParameters);  
}
```

`this.job`이 null 이니 계속 예외가 발생했습니다. 

`this.job`은 언제 초기화되나 봤더니 `setJob()`을 통해 초기화되고있었습니다.
```java
public void setJob(Job job) {  
                this.job = job;  
}
```

그러면 setJob()은 누구에 의해 호출되냐? 
`BatchTestContextBeanPostProcessor.java`
```java
@Override  
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
                if (bean instanceof JobLauncherTestUtils jobLauncherTestUtils) {  
                               this.jobProvider.ifUnique(jobLauncherTestUtils::setJob);  
                               this.jobRepositoryProvider.ifUnique(jobLauncherTestUtils::setJobRepository);  
                               this.jobLauncherProvider.ifUnique(jobLauncherTestUtils::setJobLauncher);  
                }  
                if (bean instanceof JobRepositoryTestUtils jobRepositoryTestUtils) {  
                               this.jobRepositoryProvider.ifUnique(jobRepositoryTestUtils::setJobRepository);  
                }  
                return bean;  
}
```

다음 메소드에서 ifUnique()를 통해 Job이 하나만 있는지 검증하고 그렇다면 job을 설정해주는 것이었습니다. 

만약에 두 개 이상의 job을 선언하였다면 다음과 같이 설정해야 합니다.
```java
@Qualifier("jobWithStringParameter")  
@Autowired  
private Job jobWithStringParameter;  
  
@Before  
public void setup() {  
    jobLauncherTestUtils.setJob(jobWithStringParameter);  
}
```