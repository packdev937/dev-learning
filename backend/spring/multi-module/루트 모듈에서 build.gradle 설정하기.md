### 1. 개요
각각의 모듈의 특성에 맞게 의존성을 추가하고 있지만 불가피하게 공통적으로 사용하는 의존성이 존재했습니다.
`ex) lombok, junit`

따라서 이를 루트 모듈에서 정의함으로써 공통적으로 작성하는 의존성을 제거하기로 하였습니다.

###  2. allProjects, subProjects
우선 큰 틀에서는 다음 두 분류로 나눌 수 있는데요, 각각 allProjects, subProjects 입니다. 

`allProjects`는 부모 모듈을 포함한 모든 서브 모듈에 적용하는 설정이며, 
`subProjects`는 부모 모듈을 제외한 모든 서브 모듈에 적용하는 설정입니다. 

예제 `build.gradle`
```gradle
// 모든 모듈에 적용  
allprojects {  
    apply plugin: 'java'  
  
    group = 'group`  
    version = '0.0.1-SNAPSHOT'  
  
    sourceCompatibility = '17'  
  
    repositories {  
        mavenCentral()  
    }  
}  
  
// 서브 모듈에 적용  
subprojects {  
  
    dependencies {  
        // Lombok  
        compileOnly 'org.projectlombok:lombok'  
        annotationProcessor 'org.projectlombok:lombok'  
  
        // Validation  
        implementation 'org.springframework.boot:spring-boot-starter-validation'  
        // devtools  
        implementation 'org.springframework.boot:spring-boot-devtools'  
  
        // Test  
        testImplementation 'org.springframework.boot:spring-boot-starter-test'  
        testImplementation 'io.rest-assured:rest-assured:5.3.0'  
        testImplementation 'org.junit.jupiter:junit-jupiter'  
        testImplementation platform('org.junit:junit-bom:5.9.1')  
    }  
}  
  
jar {  
    enabled = false  
}  
  
tasks.named('test') {  
    useJUnitPlatform()  
}
```