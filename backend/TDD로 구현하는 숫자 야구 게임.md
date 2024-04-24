## 1. 개요

자바지기로 유명한 박재성님의 TDD 강의를 정리해보았습니다.

## 2. TDD란?

> TFD (Test First Development) + 리팩토링

TDD**는 프로덕션 코드**를 구현하기 전에 **단위 테스트 코드**를 먼저 작성하는 것입니다.

여기에서 알 수 있는 것은 **TDD**와 **단위 테스트**는 다르다는 것입니다.

하지만 **단순히 테스트 코드를 작성**하는 것만이 아닌 **리팩토링 과정**이 필수적으로 포함되어 있습니다.

**리팩토링**은 기능에 대한 변화는 없으면서 클래스 구조, 메소드 분리 등의 설계 활동입니다.

이렇게 하게 되면 작은 단위로 기능을 추가할 때 마다 끊임없이 설계를 개선해 나갈 수 있습니다.

### TDD by Example에서는

- **TDD**란 프로그래밍 의사 결정과 피드백 사이의 간극을 의식하고 제어하는 기술이다.
- **TDD**는 아이러니 중 하나는 테스트 기술이 아니라는 점이다. **TDD**는 분석 기술이며, 설계 기술이기도 하다

**TDD**를 잘 작성하려면 TO DO LIST를 잘 작성해야 합니다. 참고로 좋은 TO DO LIST는 요구 사항 분석이 잘 된 결과 중 하나 입니다.

**TDD**를 하는 이유는 다음과 같습니다.

- 디버깅 시간을 줄여줍니다.
- 동작하는 문서 역할을 합니다.
- 변화에 대한 두려움을 줄여준다.

### **TDD의 사이클**

1. 실패하는 테스트를 구현합니다.
    
2. 테스트가 성공하도록 프로덕션 코드를 구현합니다.
    
3. 프로덕션 코드와 테스트 코드를 리팩토링 합니다.
    
    리팩토링 시 프로덕션 코드만 리팩토링 하는 것이 아닌 테스트 코드도 함께 리팩토링 합니다. 그렇지 않는다면 어느 시점에 테스트 코드에 엄청난 중복이 생길 수 있습니다.
    

### TDD 원칙

1. 실패하는 단위 테스트를 작성할 때 까지 프로덕션 코드를 작성하지 않습니다.
    
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성합니다.
    
3. 현재 실패하는 테스트를 통과할 정도만 실제 코드를 작성합니다.
    
    즉, 너무 많은 부분을 예측해서 개발하는 것이 아닌 현재 테스트 코드를 만족할 수준의 프로덕션 코드를 작성하는데 집중하는 것입니다.
    
    _(DON’T OVERENGINEERING)_
    

### 프로덕션 코드가 없는 상태에서 단위 테스트

_“어디서, 어떻게 시작해야될지 모르겠다.”, “막막하다.”_

위와 같은 감정은 당연한 감정입니다.

**도메인 지식, 객체 설계 경험이 있는 경우**

- 요구 사항 분석을 통해 대략적인 객체 설계
- UI, DB 등과 의존 관계를 가지지 않는 핵심 도메인 영역을 집중 설계

주의할 점은 TDD 라고 해서 설계를 안하는 것이 아닙니다.

View, Controller는 테스트 하기 어렵기 때문에 핵심 도메인 로직이 포함되어 있는 도메인 영역에 대한 단위 테스트를 진행합니다.

그래서 **대략적인 도메인 객체 설계**가 필요합니다.

참고로 테스트 하기 어려운 Random 값 생성 같은 경우는 격리 시키는 과정이 필요합니다.

## 3. TDD로 숫자 야구 게임 구현

TDD를 쉽게 시작하는 방법은 기능 구현 목록을 작성한 후 테스트가 가능한 부분을 도전하는 것입니다.

```java
-[ ] 1~9의 숫자 중 랜덤으로 3개의 숫자를 구한다 
-[ ] 사용자로부터 입력 받는 3개 숫자를 예외 처리 한다 
	-[ ] 1~9 사이의 숫자인가?
	-[ ] 중복 되는 숫자가 존재하는가?
	-[ ] 3자리 숫자인가?
-[ ] 위치와 숫자 값이 같은 경우는 스트라이크로 판별한다
-[ ] 위치는 다른데 숫자 값이 같다면 볼로 판별한다
-[ ] 숫자 값이 다른 경우는 낫싱으로 판별한다
-[ ] 사용자가 입력한 값에 대한 실행 결과를 구한다 
```

**1단계 - 가장 테스트 코드를 짜기 쉬운 부분은 유효성 체크 (Util 성) 기능입니다.**

- ex) 사용자로부터 입력 받는 3개 숫자를 예외 처리 한다

테스트 코드를 처음 짤 때 생각할것은 인풋과 아웃풋을 정하는 것입니다.

예를 들어, 유효성 검증 코드의 인풋과 아웃풋은 다음과 같이 정의할 수 있습니다.

- 숫자를 넘겨준다 (input)
- 불리안 값을 받는다 (output)

**2단계 - 테스트가 가능한 부분에 대해서 TDD를 작성합니다.**

- 랜덤, 시간 값처럼 무작위 값이 아닌 테스트가 가능한 부분들을 의미합니다.
- ex) - 위치와 숫자 값이 같은 경우는 스트라이크로 판별한다 - 위치는 다른데 숫자 값이 같다면 볼로 판별한다 - 숫자 값이 다른 경우는 낫싱으로 판별한다

단, 전체를 한 번에 구현하려고 하는 것은 쉽지 않습니다.

```java
PlayResult result = play(Arrays.asList(1, 2, 3), Arrays.asList(4, 5, 6));
```

우리는 프로그램을 잘개 쪼개서 구현할 필요가 있습니다.

예를 들어, 123과 456을 비교하는 것은 여러 기능이 필요할 수 있지만 하나의 숫자만 비교하는 것은 비교적 단순합니다.

```java
@Test
void nothing() {
		Ball computer = new Ball(1, 4);
		BallStatus status = computer.play(new Ball(2, 5));
		assertThat(status).isEqualTo(BallStatus.NOTHING);
}
```

TDD는 RED 상태. 즉, 안돌아가는 코드를 만든 다음에 하나씩 만들어가는 과정이 필요합니다.

`Ball.java`

```java
public class Ball {
		public Ball(int i, int i1)ㅌ {
		
		}
		
		public BallStatus play(Ball ball) {
				return BallStatus.NOTHING;
		}
}
```

`BallStatus.java`

```java
public Enum BallStatus {
		NOTHING;
}
```

아직 내부적인 기능은 작성하지 않았습니다. 하지만 위 테스트 코드는 잘 돌아갈 것입니다. 테스트가 성공하도록 코드를 짜는 단계, 그린입니다.

그 다음에 실질적인 코드를 채우며 리팩토링을 진행합니다.

`BallTest.java`

```java
public class BallTest {
		private Ball computer;
		
		@BeforeEach
		void setup(){
				computer = new Ball(1, 4);
		}
		@Test
		void 숫자와_위치가_같다면_스트라이크를_반환한다(){
				BallStatus status = computer.play(new Ball(1, 4));
				assertThat(status).isEqualTo(BallStatus.STRIKE);
		}
		
		@Test
		void 숫자는_같은데_위치가_다르다면_볼을_반환한다(){
				BallStatus status = computer.play(new Ball(2, 4));
				assertThat(status).isEqualTo(BallStatus.BALL);
		}
		
		@Test
		void 숫자가_다르다면_낫싱을_반환한다(){
				BallStatus status = computer.play(new Ball(2, 5));
				assertThat(status).isEqualTo(BallStatus.NOTHING);
		}
}
```

`Ball.java`

```java
public class Ball {
		private final int position;
		private final int number;
		
		public Ball(int position, int number) {
				this.position = position;
				this.number = number;
		}
		
		public BallStatus play(Ball ball) {
				if(this.equals(ball)){
						return BallStatus.STRIKE;
				}
		// 객체의 필드에 직접 접근하는 것 보다는 객체에 메세지를 보내라
				if(ball.matchNumber(this.number){ 
						return BallStatus.BALL;
				}
				return BallStatus.NOTHING;
		}
		
		public boolean matchNumber(int number){
				return this.number == number;
		}
}
```

**3단계 - 조금씩 복잡한 테스트를 간소화 시키자**

위 단계에서는 숫자 하나와 하나를 비교 했다면 이제는 숫자 세 개와 하나를 비교할 수 있지 않을까요?

가령 123 과 1을 비교해서 스트라이크를 판단하는 것입니다.

`Balls.java`

```java
public class Balls {
		private final List<Ball> balls;
		private static final BALLS_SIZE = 3;
		
		public Balls(List<Integer> input){
				this.balls = mapToBalls(input);
		}		
		
		public List<Ball> mapToBalls(List<Integer> input) {
				List<Ball> balls = new ArrayList<>();
				for(int i = 0; i<BALLS_SIZE; i++){
						balls.add(new Ball(i+1, input.get(i)));
				}
				return balls;
		}
		
		public BallStatus play(Ball ball) {
				return balls.stream()
								.map(answer -> answer.play(ball))
								.filter(BallStatus::isNotNothing)
								.findFirst()
								.orElse(BallStatus.NOTHING);
		}
}
```

`BallsTest.java`

```java
public class BallsTest {
		private Balls balls;
		
		@BeforeEach
		void setup(){
				balls = new Balls(Arrays.asList(1,2,3));
		}
		
		@Test
		void 숫자와_위치가_일차한다면_스트라이크를_반환한다(){
				BallStatus status = balls.play(new Ball(1,1));
				assertThat(status).isEqualTo(BallStatus.STRIKE);
		}
}
```

마지막으로 숫자 세 개와 숫자 세 개를 비교하는 로직을 진행할 수 있습니다. 그 전에 천천히 단계를 밟아왔기 때문에 그렇게 어려운 작업이 아닙니다.

`BallsTest.java`

```java
@Test
void play() {
		PlayResult result = balls.play(Arrays.asList(4,5,6));
		assertThat(result.getStrike()).isEqualTo(0);
}
```

아직 코드를 짜지 않은 상태에서 다음과 같은 구성을 생각해볼 수 있습니다.

- 세 개의 숫자에 대해서 스트라이크, 볼, 낫싱을 판단해야 하니 해당 값을 카운트 해 줄 PlayResult VO 객체를 만들자
- play()로 Ball 객체를 넘기는게 아니라 List 형태의 값을 넘기자

`Balls.play()`

```java
public PlayResult play(List<Integer> input) {
		Balls inputBalls = new Balls(input);
		PlayResult result = new PlayResult();

		for(Ball ball : balls) {
				BallStatus status = inputBalls.play(ball);
				result.report(status);
		}
		
		return result;
}
```

`PlayResult.java`

```java
@Getter
public class PlayResult {
		private int strike;
		private int ball;
		private int nothing;
		
		private final static int GAME_END_CRITERION;

		public void report(BallStatus status) {
				if(status.isStrike()){
						strike++;
				} else if(strike.isBall()) {
						ball++;
				} else if(strike.isNothing()) {
						nothing++;
				}
		}
		
		public boolean isGameEnd(){
				return this.strike == GAME_END_CRITERION;
		}
}
```

다시 한번 강조하지만 객체의 값을 가져오는 것이 아닌 객체한테 메세지를 넘기는 행위가 중요합니다.