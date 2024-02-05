문제 상황은 다음과 같습니다. 
- User 에 VO (Value Object) 객체로 Stat 클래스가 있음 
- Stat은 STR, PEA, SKI 등의 필드를 가지고 있음 
- 다음과 같은 JSON 데이터를 요청함
```
"characterType": "A",
	"stat": {
		"STR": 10,
		"SPI": 15,
		"PEA": 5,
		"SKL": 20
	}
```

하지만 제대로 된 데이터가 삽입 되지 않았는데요, 그 이유는 다음과 같습니다.

1. @Setter가 존재하지 않음 
2. 대문자 매핑으로 인한 오류가 발생함

원인 모를 이유도 있지만, JSON 변환을 강제하기 위해서 @JsonProperty를 사용할 수 있습니다. 

예시 코드는 다음과 같습니다.
```java
package com.example.sunshineserver.user.domain;  
  
import jakarta.persistence.Embeddable;  
import lombok.AccessLevel;  
import lombok.EqualsAndHashCode;  
import lombok.Getter;  
import lombok.NoArgsConstructor;  
import lombok.Setter;  
import org.springframework.data.annotation.AccessType;  
  
@Getter  
@Setter  
@Embeddable  
@EqualsAndHashCode  
@AccessType(AccessType.Type.FIELD)  
@NoArgsConstructor(access = AccessLevel.PROTECTED)  
public class Stat {  

	@JsonProperty("STR")
    private int str;  
    @JsonProperty("SPI")
    private int spi;  
    @JsonProperty("PEA")
    private int pea;  
    @JsonProperty("SKL")
    private int skl;  
  
    public Stat(int str, int spi, int pea, int skl) {  
        this.str = str;  
        this.spi = spi;  
        this.pea = pea;  
        this.skl = skl;  
    }  
  
    public static Stat of(final int STR, final int SPI, final int PEA, final int SKL) {  
        return new Stat(STR, SPI, PEA, SKL);  
    }  
}
```