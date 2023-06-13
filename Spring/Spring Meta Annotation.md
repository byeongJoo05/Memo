# Spring Meta Annotation

Spring 에서는 Annotation 사용에 대한 기능을 많이 제공하고 있다.

Annotation은 각 기능에 필요한 만큼 많은 기능을 내포하고 있으며, **이러한 내용을 개발자들이 잘 알지 못하더라도 필요한 기능만 쉽게 사용할 수 있도록 제공**되어진다.

하지만 이런 설정이 다 되어 있는 Annotation에 대해 필요없는 내용이 포함되었거나 필요로 더 사용하는 내용이 있어 커스터마이징이 필요한 경우가 있을 것이다.

그렇기에 커스텀 어노테이션을 만들기 위해 우리는 `meta-annotation`, `@Target`, `@Retention` 에 대해 개념을 알아놓을 필요가 있다.

## Meta-annotation

**meta-annotation**은 다른 annotation에서도 사용되는 annotation의 경우를 말하며 custom-annotation을 생성할 대 주로 사용된다.

`@Service` 를 뜯어보면서 더욱 공부해보자.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Spring will see this and treat @Service in the same way as @Component
public @interface Service {

// ....
}
```

@Service 는 스프링 빈으로 등록시키기 위해 `@Component` 를 내장하고 있는 형태이며, 여기서 @Component가 meta-annotation 이다.

그럼 `@Target` 과 `@Retention`은 무엇일까?

## @Target

`@Target` 은 Java Compiler 가 annotation 이 어디에 적용될지 결정하기 위해 사용한다.

사용법은 `@Target({ElementType.XXX , ElementType.XXX , …})` 처럼 XXX 부분에 자신의 어노테이션 부착을 원하는 타입으로 지정해주면 된다.

타입 종류

- CONSTRUCTOR
    - 생성자
- METHOD
    - 메서드
- FILED
    - 필드
- TYPE
    - 클래스, 인터페이스, 열거타입(ENUM)
- ANNOTATION_TYPE
    - 어노테이션
- LOCAL_VARIABLE
    - 지역변수
- PACKAGE
    - 패키지

## @Retention

`@Retention` 은 어노테이션이 실제로 적용되고 유지되는 범위이다.

Policy 에 관련된 어노테이션이기에 컴파일 이후에도 JVM 에서 참조가 가능한 RUNTIME 으로 지정한다.

종류는 다음과 같다.

- RetentionPolicy.RUNTIME
    - 컴파일 이후에도 JVM에 의해 계속 참조가 가능하다. 주로 리플렉션이나 로깅에 많이 사용된다.
- RetentionPolicy.CLASS
    - 컴파일러가 클래스를 참조할 때까지 유효하다.
- RetentionPolicy.SOURCE
    - 컴파일 전까지만 유효하다. 컴파일 이후에는 사라지게 된다.
    

## 참고 자료

[[Java] 자바 어노테이션 Annotation @Target 알아보기.](https://seeminglyjs.tistory.com/249)

[[Spring] Meta Annotation 이란?(@Target, @Retention)](https://sanghye.tistory.com/39)