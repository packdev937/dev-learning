### 1. QueryDSL이란? 
QueryDSL은 SQL, JPQL을 코드로 작성할 수 있도록 해주는 빌더 API 입니다. 조금 이른감이 있지만 예제 코드로 둘을 비교해보도록 하겠습니다.

먼저 JPQL 코드입니다. JPQL은 문자열 기반으로 쿼리를 작성하며 컴파일 타임에 타입 검사를 할 수 없습니다.
``` java
public List<Book> findBooksByTitle(String title, EntityManager em) {
    String jpql = "SELECT b FROM Book b WHERE b.title = :title";
    return em.createQuery(jpql, Book.class)
            .setParameter("title", title)
            .getResultList();
}
```

QueryDSL 코드입니다. 동일한 기능을 타입 세이프한 방식으로 구현할 수 있습니다. 이는 컴파일 시점에 쿼리의 오타나 타입 불일치 등의 문제를 감지할 수 있게 해줍니다.
```java
import static com.myapp.domain.QBook.book;

public List<Book> findBooksByTitleQueryDSL(String title, JPAQueryFactory queryFactory) {
    return queryFactory.selectFrom(book)
            .where(book.title.eq(title))
            .fetch();
}

```
참고로 QueryDSL은 Entity 클래스와 매핑되는 QClass 객체를 사용해서 쿼리를 실행합니다.

다시 한번 강조하자면 QueryDSL은 도메인 타입의 프로퍼티를 반영해서 생성한 쿼리 타입을 이용해서 쿼리를 작성하게 됩니다. 이는 JPQL이나 SQL에서 String을 통해 작성된 오타 등으로 잘못된 참조를 하게 될 가능성을 막아주는데 큰 의미를 두고 있습니다. 

#### 1-1. QClass란
QClass란 Entity 클래스의 메타 데이터를 가지고 있는 클래스 입니다. 많은 분들이 의구심을 표한 것과 마찬가지로 저 또한 *"Entity 클래스를 사용하지 않고 굳이 QClass를 만들어서 사용하는가"* 에 대한 의문이 있었습니다.

우선 JPA_APT(JPAAnnotationProcessorTool)가 @Entity와 같은 특정 어노테이션을 찾고 해당 클래스를 분석하여 QClass를 생성합니다.

``` bash
# @Entity 어노테이션을 선언한 클래스를 탐색하고 Q 클래스를 생성 
annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa"
```

메타 정보를 담고 있는 QClass를 이용하면 타입 안정성을 보장하며 쿼리를 작성할 수 있게 됩니다. 
즉, 
1) QClass를 사용하여 쿼리를 작성한다면 엔티티 속성을 직접 참조하여 쿼리를 구상할 수 있으며, 
2) 속성의 타입을 정확하게 표현함으로 타입에 맞지 않는 연산이나 비교를 감지할 수 있고, 
3) 이를 통해 컴파일 시점에 오류를 확인할 수 있으며 
4) IDE 자동 완성 기능을 사용할 수 있습니다. 

### 2. [환경 설정](https://velog.io/@soyeon207/QueryDSL-Spring-Boot-%EC%97%90%EC%84%9C-QueryDSL-JPA-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
`build.gradle`
```bash
dependencies {
	# QueryDSL을 사용하기 위한 라이브러리
	implementation 'com.querydsl:querydsl-jpa'
	
	# @Entity 어노테이션을 선언한 클래스를 탐색하고 Q 클래스를 생성 
	annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"  

	# java -> jakarta
	annotationProcessor 'jakarta.persistence:jakarta.persistence-api'
	annotationProcessor 'jakarta.annotation:jakarta.annotation-api'
}

# gradle 빌드 시 QClass 소스도 함께 빌드하기 위해서 sourceSets에 해당 위치를 추가
	def querydslSrcDir = 'src/main/generated'
	sourceSets {
	  main {
	    java {
	      srcDirs += [ querydslSrcDir ]
	    }
	  }
	}
	
	compileJava {
	    options.compilerArgs << '-Aquerydsl.generatedAnnotationClass=javax.annotation.Generated'
	}
	
	tasks.withType(JavaCompile) {
		options.generatedSourceOutputDirectory = file(querydslSrcDir)
	}
	
	# build clean 시 생성되었던 QClass를 모두 삭제
	clean {
	  delete file(querydslSrcDir)
	}
```

- QClass를 삭제하기 위해서는 gradle > build > clean 
- QClass를 만들기 위해서는 gradle > build > compileJava

### 3. 사용 예제



### Reference
https://dev.gmarket.com/33