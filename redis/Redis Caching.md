## 1. Redis란
> 대표적인 인메모리 키-값 데이터 저장소으로서 오픈 소스 기반의 비관계형 데이터 베이스 입니다.Redis는 캐싱, 세션 관리, 메세지 브로커 등의 다양한 용도로 사용됩니다.

이번 포스팅에서는 Redis를 통해 캐싱을 하는 과정에 대해 서술해보도록 하겠습니다. 
### 1-1. 캐시란? 
캐시는 데이터를 일시적으로 저장하는 데이터 공간으로 효율적인 데이터베이스 활용을 위해 사용합니다. 
자주 사용되는 데이터를 캐시에 적재하여 해당 데이터를 조회할 때 마다 원본 데이터를 조회하는 대신 캐시에 접근하여 빠르게 데이터를 조회할 수 있도록 합니다. 
## 2. 설정하기 
Redis를 사용하기 위해서 다음의 의존성을 추가해줍니다. 
`build.gradle`
```yaml
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```
해당 라이브러리에는 다양한 Redis Client들이 포함되어 있습니다. 

### 2-1. Redis Client가 뭐야? 
Redis 클라이언트는 애플리케이션이 Redis 서버와 상호작용할 수 있도록 해주는 라이브러리입니다. 

위에서 언급했듯 애플리케이션 측에서 실행되며 Redis 서버에 명령을 보내고 응답을 받습니다. 

서버는 해당 요청을 받고 데이터 CRUD 등의 기능을 제공합니다. 

대표적인 라이브러리로는 Jedis와 Lettuce가 있습니다. 

`Jedis`의 특징으로는 동기식 API, 직관적인 API 설계, 높은 성능, 다중 스레드 환경에서 안전하지 않습니다.

바반면 `Lettuce`는 비동기 및 동기 API를 모두 지원하는 라이브러리입니다. 다중 스레드 환경에서 안전하며 Redis Cluster를 지원하거나 고성능 비동기 I/O를 제공합니다. 

Lettuce를 사용해서 설정을 진행해보겠습니다.
`RedisConfig.java`
```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory();
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());
        // 키를 직렬화하는 방식을 설정합니다.
        template.setKeySerializer(new StringRedisSerializer());

		// 값을 직렬화하는 방식을 설정합니다.
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());

		// 해시 키를 직렬화하는 방식을 설정합니다. 
		template.setHashKeySerializer(new StringRedisSerializer()); 
		
		// 해시 값을 직렬화하는 방식을 설정합니다. 
		template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
```

이렇게 만든 `RedisTemplate`이 Redis 서버와의 통신하는 역할을 합니다. 

### 2-2. @RedisHash
`@RedisHash` 어노테이션을 사용하면 Spring Data Redis가 해당 엔티티를 자동으로 Redis 해시에 저장하고 관리해줍니다.

간단한 예제를 작성해보겠습니다. 핵심 코드만 작성하기 때문에 생략되는 코드가 존재할 수 있습니다. 
```java
@RedisHash(value = "sport", timeToLive = 300) // 300초 동안 TTL 설정
public class Sport {
    @Id
    private String id;
    private String name;
    private String description;
    ...
}

@Repository
public interface SportRepository extends CrudRepository<Sport, String> {}

@Service
public class SportService {
	public void saveSport(Sport sport) {
		sportRepository.save(sport
	} 
	public Sport findSportById(String id) { 
		return sportRepository.findById(id).orElse(null); 
	}
}
```

`@RedisHash` 어노테이션의 `timeToLive` 속성을 사용하면, 해당 엔티티가 Redis에 저장될 때 자동으로 TTL이 설정됩니다. 위 예시에서는 `Sport` 엔티티가 Redis에 저장된 후 300초(5분) 동안 유효하며, 그 이후에는 자동으로 삭제됩니다.

## 3. 캐싱 서버로서의 Redis
코드를 통해 설명해보겠습니다. 
참고로 밑에 코드는 Cache 의존성을 별도로 추가한 상태입니다.

`build.gradle`
```java
implementation 'org.springframework.boot:spring-boot-starter-cache'
```

`application.yml`
```yml
spring:
  data:
    redis:
      host: 127.0.0.1
      port: 6379
```

`RedisConfig.java`
```java
@Configuration
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(
                new RedisStandaloneConfiguration(host, port)
        );
    }
}
```

`CacheConfig.java`
```java
@EnableCaching
@Configuration
public class CacheConfig {

    @Bean
    public RedisCacheConfiguration redisCacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
		        // 캐시 만료 시간
                .entryTtl(Duration.ofSeconds(60))
                // 캐싱할 때 null 값을 허용 안함
                .disableCachingNullValues()
                // Key를 직렬화 할 때 사용하는 규칙
                .serializeKeysWith(
	                RedisSerializationContext
		                .SerializationPair
		                .fromSerializer(new StringRedisSerializer())
                )
                // Value를 직렬화 할 때 사용하는 규칙 
                .serializeValuesWith(
                        RedisSerializationContext
                        .SerializationPair
                        .fromSerializer(new GenericJackson2JsonRedisSerializer())
                );
    }

    @Bean
    public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
        return (builder) -> builder
		        // 특정 캐시에 대한 개별 설정을 정의 
                .withCacheConfiguration("cache1",
		                // 기본 캐시 설정을 가져옵니다.
                        RedisCacheConfiguration.defaultCacheConfig()
		                    // 캐시 이름에 접두사를 추가하는 방법을 정의합니다.
                                .computePrefixWith(
                                // prefix::cache1::
                                cacheName -> "prefix::" + cacheName + "::")
                                // 캐시를 120초로 설정합니다
                                .entryTtl(Duration.ofSeconds(120))
                                // 중복임으로 제거 가능합니다. 
                                .disableCachingNullValues()
                                .serializeKeysWith(
                                        RedisSerializationContext
                                        .SerializationPair
                                        .fromSerializer(new StringRedisSerializer())
                                )
                                .serializeValuesWith(
                                        RedisSerializationContext
                                        .SerializationPair
                                        .fromSerializer(new GenericJackson2JsonRedisSerializer())
                                ))
                .withCacheConfiguration("cache2",
                        RedisCacheConfiguration.defaultCacheConfig()
                                .entryTtl(Duration.ofHours(2))
                                .disableCachingNullValues());
    }
}
```

Spring Data Redis 를 사용한다면 Spring Boot 가 `RedisCacheManager` 를 자동으로 설정해줍니다.

참고로 직접 구현한 `RedisCacheManager`는 다음과 같습니다.
```java
@Bean 
public RedisCacheManager cacheManager(
	RedisConnectionFactory factory, 
	RedisCacheConfiguration redisCacheConfiguration) {
	 
	return RedisCacheManager.builder(factory)
		.cacheDefaults(redisCacheConfiguration)
		.withInitialCacheConfigurations(redisCacheManagerBuilderCustomizer()
		.getCacheConfigurations()) 
		.build(); 
}
```

`RedisCacheManager`는 Spring Framework에서 제공하는 캐시 관리자 중 하나로 Redis를 백엔드 캐시 저장소로 사용하여 캐시를 관리하는 역할을 합니다. 

주요 역할로는 캐시를 생성 및 관리하며 캐시 설정을 적용하고 캐시 연산 처리 및 초기화 등의 작업을 합니다.
코드를 보시면 앞서 설정했던 `RedisConnectionFactory`, `RedisCacheConfiguration`, 그리고 `RedisCacheManagerBuilderCustomizer` 등을 속성 값으로 넘겨주는 것을 확인할 수 있습니다.

**RedisCacheConfiguration**:
    기본 캐시 설정을 정의합니다. 모든 캐시에 적용될 기본 TTL(5분), null 값 캐싱 비활성화, 문자열 키 직렬화, JSON 값 직렬화 등을 포함합니다.

**RedisCacheManagerBuilderCustomizer**:
    여러 캐시에 대해 개별 설정을 정의합니다. 각 캐시마다 다른 TTL 및 직렬화 방법을 설정할 수 있습니다.
    예를 들어, `cache1` 캐시는 TTL이 60초로 설정되고, `cache2` 캐시는 TTL이 1시간으로 설정됩니다.

작동 방식은 다음과 같습니다. 
- 애플리케이션에서 `@Cacheable`, `@CachePut`, `@CacheEvict` 등의 어노테이션을 사용하여 캐시를 요청하면, `RedisCacheManager`가 해당 요청을 처리합니다.
- 캐시 요청이 들어오면 `RedisCacheManager`는 캐시 이름을 기반으로 해당 캐시를 찾고, 캐시 설정을 적용하여 데이터를 처리합니다.
- 데이터를 Redis에 저장하거나 조회할 때, 설정된 직렬화 방식을 사용하여 데이터를 직렬화(Serialize)하거나 역직렬화(Deserialize)합니다.
- 각 캐시 항목에 대해 설정된 TTL을 적용합니다. TTL이 만료되면 해당 캐시 항목은 자동으로 삭제됩니다.


하지만 Redis 는 직렬화/역직렬화 때문에 별도의 캐시 설정이 필요하고 이 때 사용하는게 `RedisCacheConfiguration` 입니다.

`RedisCacheConfiguration` 설정은 Redis 기본 설정을 오버라이드 한다고 생각하면 됩니다.

- `computePrefixWith`: Cache Key prefix 설정
- `entryTtl`: 캐시 만료 시간
- `disableCachingNullValues`: 캐싱할 때 null 값을 허용하지 않음 (`#result == null` 과 함께 사용해야 함)
- `serializeKeysWith`: Key 를 직렬화할 때 사용하는 규칙. 보통은 String 형태로 저장
- `serializeValuesWith`: Value 를 직렬화할 때 사용하는 규칙. Jackson2 를 많이 사용함

  

만약 캐시이름 별로 여러 세팅을 하고 싶다면 `RedisCacheManagerBuilderCustomizer` 를 선언해서 사용할 수 있습니다.

위 코드에서는 `cache1`, `cache2` 두 가지 캐시를 설정했으며 만약 다른 이름의 캐시를 사용하려 한다면 기본 설정인 `RedisCacheConfiguration` 를 따라갑니다.

`GenericJackson2JsonRedisSerializer` 를 사용할 때 주의할 점은 여러개의 데이터를 한번에 저장할 때 `List` 를 사용하지 말고 별도의 일급 컬렉션을 선언해서 사용해야 합니다.

### 3-1. 사용 예제
```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    @Cacheable(value = "cache1")
    public String getDataFromCache1(String param) {
        return "Data from cache1 for " + param;
    }

    @Cacheable(value = "cache2")
    public String getDataFromCache2(String param) {
        return "Data from cache2 for " + param;
    }
}
```

## Reference
- https://bcp0109.tistory.com/386
- https://velog.io/@juhyeon1114/Spring-boot%EC%97%90%EC%84%9C-Redis%EB%A1%9C-%EC%BA%90%EC%8B%B1%ED%95%98%EA%B8%B0-w.-JPA