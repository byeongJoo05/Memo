# JUnit

## given/when/then 패턴

1개의 단위 테스트를 3가지 단계로 나누어 처리하는 패턴

- given(준비) : 어떠한 데이터가 준비되었을 때
- when(실행) : 어떠한 함수를 실행하면
- then(검증) : 어떠한 결과가 나와야한다.

### 테스트 코드 작성 공통 규칙

```java
@DisplayName("로또 번호 갯수 테스트")
@Test
void lottoNumberSizeTest() {
    // given

    // when

    // then
}
```

- `@Test` : 해당 메서드가 단위 테스트임을 명시하는 어노테이션
JUnit은 테스트 패키지 하위의 @Test 어노테이션이 붙은 메서드를 단위 테스트로 인식하여 실행시킨다.
- `@DisplayName` : 메서드 네임을 읽기 편하게 사용자 입맛대로 바꾸어 테스트 결과창에서 띄워주는 어노테이션

## given-when-then 패턴을 활용한 테스트코드 예시

### 로또 자바 코드

```java
public class LottoNumberGenerator {

    public List<Integer> generate(final int money) {
        if (!isValidMoney(money)) {
            throw new RuntimeException("올바른 금액이 아닙니다.");
        }
        return generate();
    }

    private boolean isValidMoney(final int money) {
        return money == 1000;
    }

    private List<Integer> generate() {
        return new Random()
                .ints(1, 45 + 1)
                .distinct()
                .limit(6)
                .boxed()
                .collect(Collectors.toList());
    }

}
```

### 로또 번호 갯수 테스트

```java
@DisplayName("로또 번호 갯수 테스트")
@Test
void lottoNumberSizeTest() {
    // given
    final LottoNumberGenerator lottoNumberGenerator = new LottoNumberGenerator();
    final int price = 1000;

    // when
    final List<Integer> lotto = lottoNumberGenerator.generate(price);

    // then
    assertThat(lotto.size()).isEqualTo(6);
}
```

`isEqualTo()` 를 통하여 행위값과 예측값이 맞는지 확인을 한다.

### 로또 번호 범위 테스트

```java
@DisplayName("로또 번호 범위 테스트")
@Test
void lottoNumberRangeTest() {
    // given
    final LottoNumberGenerator lottoNumberGenerator = new LottoNumberGenerator();
    final int price = 1000;

    // when
    final List<Integer> lotto = lottoNumberGenerator.generate(price);

    // then
    assertThat(lotto.stream().allMatch(v -> v >= 1 && v <= 45)).isTrue();
}
```

로또 숫자가 1 ~ 45 사이의 숫자인지 boolean 값으로 검사하므로, AssertJ의 `isTrue()` 문법이 사용되었다.

이 외에도 `isFalse()`, `isNull()`, `isNotNull()` 등의 진위 여부의 관한 문법이 존재한다.

### 잘못된 로또 금액 테스트

```java
@DisplayName("잘못된 로또 금액 테스트")
@Test
void lottoNumberInvalidMoneyTest() {
    // given
    final LottoNumberGenerator lottoNumberGenerator = new LottoNumberGenerator();
    final int price = 2000;

    // when
    final RuntimeException exception = assertThrows(RuntimeException.class, () -> lottoNumberGenerator.generate(price));

    // then
    assertThat(exception.getMessage()).isEqualTo("올바른 금액이 아닙니다.");
}
```

예외 발생인 경우 when 절에서 `assertThrows()`로 감싸서 처리해야한다.

이전 코드들과 다르게 금액을 2000원으로 변경하였고, 실행하는 메소드를 assertThrows()로 감싸 주었다.

간단한 자바 애플리케이션이라서 어떤 메소드가 다른 객체와 메세지를 주고 받을 필요가 없는 경우라면 단위 테스트 작성이 간단하다. 하지만 일반적인 애플리케이션은 상당히 복잡하고, 여러 객체들이 메세지를 주고 받는다.

## Mockito

- Mockito란 개발자가 동작을 직접 제어할 수 있는 가짜 객체를 지원하는 테스트 프레임워크이다.
- 웹 애플리케이션 개발에서 생기는 여러 객체들 간의 의존성을 해결하기 위해 이 Mockito 라이브러리를 활용하여 가짜 객체를 주입시킨다.

## Mockito 사용법

### Mock 객체 의존성 주입

Mockito에서 가짜 객체의 의존성 주입을 위해서는 크게 3가지 어노테이션이 사용된다.

- `@Mock` : 가짜 객체를 만들어 반환해주는 어노테이션
- `@Spy` : Stub하지 않은 메서드들은 원본 메서드 그대로 사용하는 어노테이션
- `@InjectMocks` : `@Mock` 또는 `@Spy`로 생성된 가짜 객체를 자동으로 주입시켜주는 어노테이션

Ex) UserController에 대한 단위 테스트를 작성하고자 할 때, UserService를 사용하고 있다면 `@Mock` 어노테이션을 통해 가짜 UserService를 만들고, `@InjectMocks`를 통해 UserController에 이를 주입시킬 수 있다.

### Stub로 결과 처리

앞서 설명하였듯, 의존성이 있는 객체는 가짜 객체를 주입하여 어떤 결과를 반환하라고 정해진 답변을 준비시켜야 한다. Mockito에서는 다음과 같은 stub 메소드를 제공한다.

- `doReturn()` : 가짜 객체가 특정한 값을 반환해야 하는 경우
- `doNothing()` : 가짜 객체가 아무 것도 반환하지 않는 경우(void)
- `doThrow()` : 가짜 객체가 예외를 발생시키는 경우

### Mockito와 JUnit 결합

Mockito도 테스팅 프레임워크이기 때문에 JUnit과 결합되기 위해서는 별도의 작업이 필요하다. 기존의 JUnit4에서 Mockito를 활용하기 위해서는 클래스 어노테이션으로 `@RunWith(MockitoJUnitRunner.class)`를 붙여주어야 연동이 가능했다. 하지만 SpringBoot 2.2.0부터 공식적으로 JUnit5를 지원함에 따라, 이제부터는 `@ExtendWith(MockitoExtension.class)`를 사용해야 결합이 가능하다.

## 관련 자료

[[Spring] JUnit과 Mockito 기반의 Spring 단위 테스트 코드 작성법 (3/3)](https://mangkyu.tistory.com/145)

[통합 테스트 말고 단위 테스트만 하고 싶으면 어떻게 하죠?](https://velog.io/@mindfulness_22/unit-test-code-first-experience)