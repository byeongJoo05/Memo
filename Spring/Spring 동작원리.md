# Spring 동작원리

### 주저리주저리

저번까지 Spring의 3대 요소인 IoC, DI, AOP에 대해 깊게 알아봤다.

다시 복습하는 의미에서 3대 요소를 정리하자면 IoC(제어의 역전)는 개발자가 코드들 혹은 객체들을 엮어서 개발을 하는 것이 아닌 스프링에서 컨테이너 혹은 오브젝트(=빈)으로 서로 소스코드를 참고하여 읽어서 애플리케이션 자체적으로 내부 동작하여 개발자가 몰라도 이미 시스템상으로 작동하는 것이다. DI(의존성 주입)는 IoC와 비슷하게 한 오브젝트가 작동하기 위해 다른 오브젝트를 의존하게 되는 것을 말한다. AOP(관점지향 프로그래밍)는 우리가 로깅이나 트랜잭션처럼 다른 비즈니스 로직 + (공통으로 사용될 무언가) 를 사용할 때 쓰는 방식을 말하는데, 예를 들어 A + 로깅, B + 로깅 할 때 로깅만 따로 빼서 다른 비즈니스 로직 + 공통 관심사를 사용할 때 유용하게 할 수 있다. 이렇게 하면 관심사 분리가 될 수 있기에 더욱 효율적 로직 작성을 할 수 있다.

이와 같이 Spring의 요소에 대해 알아볼 수 있었다. 이제 우리는 스프링에 더욱 직접적인 동작원리에 대해 알아봐야한다. 일단 스프링 컨테이너부터 알아보고 가자!

## 스프링 컨테이너

### 스프링 컨테이너(Spring Container)란?

- 스프링 프레임워크의 핵심 컴포넌트
- 자바 객체의 생명 주기를 관리하며, 생성된 자바 객체들에게 추가적 기능을 제공
- 스프링에선 자바 객체를 빈(Bean) 이라고 함

스프링 컨테이너는 내부에 존재하는 빈의 생명주기를 관리하며, 생성된 빈에게 추가적 기능을 제공하는 것이다.

스프링 컨테이너는 XML 혹은 어노테이션 기반의 자바 설정 클래스로 구축할 수 있다.

그럼 왜 스프링 컨테이너를 사용해야 할까?

### 스프링 컨테이너 사용 이유

객체를 생성하기 위해서는 new 생성자를 이용해 생성해야 하며, 이런 객체 생성으로 인해 애플리케이션에서는 수많은 객체가 존재하며 서로 참조하게 된다.

객체 간 참조가 많아질수록 객체끼리는 의존성이 높아지게 되는데, 이렇게 된다면 낮은 결합도와 높은 캡슐화를 지향하는 객체지향 프로그래밍의 핵심과 먼 방식이 되어버린다.

이런 문제점을 해결하기 위해 **객체 간 의존성을 낮추어 결합도는 낮추고, 높은 캡슐화를 위해 스프링 컨테이너가 사용**하게 된다.

그렇다면 이런 스프링 컨테이너는 무엇으로 만들어져 있을까?

![img1 daumcdn](https://github.com/byeongJoo05/Memo/assets/84984586/b4b2d6ec-2127-4af3-9318-6c2e6df2dad1)
![img1 daumcdn 1](https://github.com/byeongJoo05/Memo/assets/84984586/a58a05ad-cec8-406a-a540-51c7ffa5356c)

**스프링 컨테이너는 BeanFactory와 ApplicationContext 두 종류의 인터페이스로 구현**되어 있다.

- BeanFactory
    - 빈을 생성하고 의존관계를 설정하는 기능을 담당하는 가장 기본적인 IoC 컨테이너이자 클래스
- ApplicationContext
    - ApplicationContext는 BeanFactory를 구현하고 있어 BeanFactory의 확장된 버전

> BeanFactory 라고 말할 때는 **빈을 생성하고 관계를 설정하는 IoC의 기본 기능에 초점을 맞춘 것**이며, ApplicationContext 는 **별도의 정보를 참고해서 빈의 생성, 관계 설정 등의 제어를 총괄하는 것에 초점을 맞춘 것**이다.
> 

그렇다면 이런 스프링 컨테이너가 빈을 생성하는 방식은 무엇일까?

### 빈 생성 방식 - 싱글톤 레지스트리

스프링은 기본적으로 별다른 설정을 하지 않는다면 내부에서 생성하는 빈 오브젝트를 모두 싱글톤으로 만든다.

객체를 한 번 생성해두면 애플리케이션이 종료될 때까지 추가적인 생성 작업이 필요 없는 것을 **싱글톤 방식**이라 하며, 오브젝트 생성 측면에서 자원 소모를 효율적으로 할 수 있다.

이러한 싱글톤 방식도 여러 단점이 있다. 서술해보자면 다음과 같다.

**단점**

- 클래스 밖에서는 오브젝트를 생성하지 못하도록 생성자를 private로 만든다. 이렇기에 **객체지향의 장점인 상속과 다형성을 적용할 수 없다.**
- 생성된 싱글톤 오브젝트를 저장할 수 있는 자신과 같은 타입의 스태틱 필드를 정의한다. 하지만 싱글톤은 사용하는 클라이언트가 정해져 있지 않으며, 아무 객체나 자유롭게 접근하고 수정하고 공유할 수 있는 **전역 상태를 갖는 것은 객체지향 프로그래밍에서는 권장되지 않는 프로그래밍 모델**이다.

위 2가지 이외에도 여러 단점이 존재하지만, 개인적으로 쉽게 이해할 수 있는 단점으로는 이 두 가지가 대표적이라고 본다.

기존 싱글톤 패턴의 단점때문에 스프링에서는 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공한다. 이를 **싱글톤 레지스트리(Singleton Registry)** 라고 한다.

싱글톤 레지스트리는 다음과 같은 편의성을 제공한다.

- 스태틱 메서드와 private 생성자를 사용해야 하는 비정상적인 클래스가 아니라 평범한 자바 클래스를 싱글톤으로 활용하게 해준다.
- public 생성자를 구현할 수 있기에 생성자 파라미터로 의존관계 주입이 가능하며, 테스트 시 mock 객체 생성이 가능하다.
- **싱글톤 패턴과 달리 스프링이 지지하는 객체지향적인 설계 방식과 원칙, 디자인 패턴 등을 적용하는 데 아무런 제약이 없다는 점이다.**

위와 같은 스프링 동작원리를 통하여 우리가 소스코드를 작성하며 개발해온 프로젝트 백그라운드에 스프링 컨테이너가 어떻게 동작하고 있는지 알게 되었다.

정리를 해보자면 다음과 같다.

- 스프링 컨테이너는 빈의 생명주기를 관리하며, 생성된 빈에게 추가적 기능을 제공해준다.
- 스프링 컨테이너를 사용하는 이유는 객체 간 의존성을 낮추어 느슨한 결합을 형성시키는 것과 높은 캡슐화를 사용하기 위해서이다.
- 스프링 컨테이너는 BeanFactory와 ApplicationContext 두 인터페이스로 구현되어 있는데, BeanFactory는 bean의 생성과 IoC 관련 기능을 제공하는 역할을 하며, ApplicationContext는 BeanFactory가 제공하는 기능 이외에도 부가적 기능을 추가하여 작동한다. 즉, ApplicationContext는 BeanFactory의 기능을 상속받았다는 것을 알 수 있다!
- 스프링 컨테이너는 싱글톤 레지스트리 방식을 사용한다.

이렇게 Bean들을 능숙능란하게 사용하는 스프링이였다. 

웹개발자로서 MVC패턴에 사용되는 `@Controller`, `@Repository`, `@Service` 어노테이션들을 한 번도 뜯어본 적이 없는데, 이번에 이 MVC에 사용되는 어노테이션의 동작원리까지 한 번 봐보고 가자

## MVC에 사용되는 어노테이션 뜯어보기

순서는 `@RestController` → `@Controller` → `@Service` → `@Repository` 순으로 뜯어보자.

### @RestController

```java
/**
 * A convenience annotation that is itself annotated with
 * {@link Controller @Controller} and {@link ResponseBody @ResponseBody}.
 * <p>
 * Types that carry this annotation are treated as controllers where
 * {@link RequestMapping @RequestMapping} methods assume
 * {@link ResponseBody @ResponseBody} semantics by default.
 *
 * <p><b>NOTE:</b> {@code @RestController} is processed if an appropriate
 * {@code HandlerMapping}-{@code HandlerAdapter} pair is configured such as the
 * {@code RequestMappingHandlerMapping}-{@code RequestMappingHandlerAdapter}
 * pair which are the default in the MVC Java config and the MVC namespace.
 *
 * @author Rossen Stoyanchev
 * @author Sam Brannen
 * @since 4.0
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 * @since 4.0.1
	 */
	@AliasFor(annotation = Controller.class)
	String value() default "";

}
```

RestController는 자체적으로 @Controller와 @ResponseBody를 내장하고 있는 어노테이션이다.

@Controller 를 내장하고 있기에 컴포넌트를 공유하여 사용하고 있으며, Controller Bean을 자동적으로 탐지하여 Controller 어노테이션을 사용할 수 있는 것이다.

@Target 어노테이션 내 파라미터가 TYPE 타입 이기에 사용될 곳의 타입을 선언해주면 자동 타겟팅을 해줘서 사용할 수 있다.

@Retention을 통해 우리가 알 수 있는 정도는 RUNTIME 시 이 어노테이션이 작동한다는 것이다.

### @Controller

```java
/**
 * Indicates that an annotated class is a "Controller" (e.g. a web controller).
 *
 * <p>This annotation serves as a specialization of {@link Component @Component},
 * allowing for implementation classes to be autodetected through classpath scanning.
 * It is typically used in combination with annotated handler methods based on the
 * {@link org.springframework.web.bind.annotation.RequestMapping} annotation.
 *
 * @author Arjen Poutsma
 * @author Juergen Hoeller
 * @since 2.5
 * @see Component
 * @see org.springframework.web.bind.annotation.RequestMapping
 * @see org.springframework.context.annotation.ClassPathBeanDefinitionScanner
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";

}
```

위 RestController 어노테이션과 비슷한 형상을 띄우고 있지만 달라진 점은 `@ResponseBody` 가 빠져있다는 것이다.

또한 이 Controller부터 `@Component`가 적용된 것을 볼 수 있는데, 컴포넌트가 적용되어 있기에 스프링 컨테이너가 이 어노테이션을 자동 주입할 수 있는 것이다.

Controller 어노테이션도 Target은 타입 선언문이며, 런타임시점에서 작동한다는 것을 알 수 있다.

### @Service

```java
/**
 * Indicates that an annotated class is a "Service", originally defined by Domain-Driven
 * Design (Evans, 2003) as "an operation offered as an interface that stands alone in the
 * model, with no encapsulated state."
 *
 * <p>May also indicate that a class is a "Business Service Facade" (in the Core J2EE
 * patterns sense), or something similar. This annotation is a general-purpose stereotype
 * and individual teams may narrow their semantics and use as appropriate.
 *
 * <p>This annotation serves as a specialization of {@link Component @Component},
 * allowing for implementation classes to be autodetected through classpath scanning.
 *
 * @author Juergen Hoeller
 * @since 2.5
 * @see Component
 * @see Repository
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";

}
```

`@Service` 는 DDD 디자인 패턴을 적용한 어노테이션이란 것을 도큐먼트를 보고 알 수 있었다.

이 어노테이션은 타입 선언 시 적용될 수 있으며, 런타임 발생 시 Bean으로 등록된다는 것을 알 수 있다.

### @Repository

```java
/**
 * Indicates that an annotated class is a "Repository", originally defined by
 * Domain-Driven Design (Evans, 2003) as "a mechanism for encapsulating storage,
 * retrieval, and search behavior which emulates a collection of objects".
 *
 * <p>Teams implementing traditional Jakarta EE patterns such as "Data Access Object"
 * may also apply this stereotype to DAO classes, though care should be taken to
 * understand the distinction between Data Access Object and DDD-style repositories
 * before doing so. This annotation is a general-purpose stereotype and individual teams
 * may narrow their semantics and use as appropriate.
 *
 * <p>A class thus annotated is eligible for Spring
 * {@link org.springframework.dao.DataAccessException DataAccessException} translation
 * when used in conjunction with a {@link
 * org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor
 * PersistenceExceptionTranslationPostProcessor}. The annotated class is also clarified as
 * to its role in the overall application architecture for the purpose of tooling,
 * aspects, etc.
 *
 * <p>As of Spring 2.5, this annotation also serves as a specialization of
 * {@link Component @Component}, allowing for implementation classes to be autodetected
 * through classpath scanning.
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @since 2.0
 * @see Component
 * @see Service
 * @see org.springframework.dao.DataAccessException
 * @see org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";
```

`@Repository` 는 DDD 디자인 패턴을 적용한 어노테이션이란 것을 도큐먼트를 보고 알 수 있었다.

이 어노테이션은 타입 선언 시 적용될 수 있으며, 런타임 발생 시 Bean으로 등록된다는 것을 알 수 있다.

이처럼 MVC에서 사용되는 대표적인 어노테이션들은 우리가 컴포넌트 등록을 더욱 편히하기 위해 만들어진 것들이란 사실을 알 수 있었다.

생각해보면, 위와 같은 어노테이션이 등록된 클래스 내부에 구체클래스를 작성하여 사용한 경험은 백엔드 개발을 했다면 누구나 경험해봤을 것이다.

## 참고 자료

[[Spring] 스프링 컨테이너(Spring Container)란 무엇인가?](https://ittrue.tistory.com/220)

[스프링 컨테이너(BeanFactory, ApplicationContext)](https://beststar-1.tistory.com/39)

[[Spring] Meta Annotation 이란?(@Target, @Retention)](https://sanghye.tistory.com/39)