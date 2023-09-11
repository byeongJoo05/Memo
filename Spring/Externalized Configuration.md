# Externalized Configuration

## Externalized Configuration

스프링 부트에선 설정을 외부로 뺄 수 있어 같은 애플리케이션을 서로 다른 환경으로 작업할 수 있다. 외부 설정은 자바 Properties 파일, YAML 파일, 환경 변수, 커맨드라인 인자 등 다양한 소스를 활용할 수 있다.

프로퍼티 값은 `@Value` 어노테이션을 통해 빈에 직접 주입하거나, 스프링의 `Environment` 인터페이스로 접근해도 되고, `@ConfigurationProperties`를 통해 객체 구조에 바인딩할 수도 있다.

스프링 부트는 `PropertySource`를 적용하는 순서를 정확하게 설계해놨기 때문에, 사리에 맞게 값을 재정의할 수 있다. 프로퍼티를 적용하는 순서는 다음과 같다 (우선순위는 낮은 순에서 높은 순으로 오버라이드되므로, 공통으로 사용할 값일수록 우선순위가 낮아지고, 프로파일이나 특정 환경별로 사용할 값일수록 우선순위가 높아진다). 

- 우선순위

| No | Priority | Property |
| --- | --- | --- |
| 1 | Low | 기본 속성 (SpringApplication.setDefaultProperties) |
| 2 |  | @Configuration 클래스 |
| 3 |  | 컨피그 데이터 (application.properties 파일) |
| 4 |  | OS 환경 변수 |
| 5 |  | 자바 시스템 변수 (System.getProperties()) |
| 6 |  | JNDI attributes (java:comp/env) |
| 7 |  | ServletContext 초기화를 위한 매개변수 |
| 8 |  | ServletConfig 초기화를 위한 매개변수 |
| 9 |  | SPRING_APPLICATION_JSON (환경 변수 또는 시스템 변수에 포함된 인라인 JSON) |
| 10 |  | Command Line Arguments |
| 11 |  | @SpringBootTest와 함께 사용된 properties |
| 12 |  | @TestPropertySource |
| 13 | High | devtools 활성 상태일 때 $HOME/.config/spring-boot 디렉토리 내 devtools 전역 설정 |

3번 컨피그 데이터 파일은 다음과 같은 순서로 적용된다.

1. jar에 패키징한 애플리케이션 프로퍼티 (`application.properties` 또는 YAML로 작성한 파일)
2. jar에 패키징한 프로파일 전용 애플리케이션 프로퍼티 (`application-{profile}.properties` 또는 YAML로 작성한 파일)
3. 패키징한 jar 밖에 있는 애플리케이션 프로퍼티 (`application.properties` 또는 YAML로 작성한 파일)
4. 패키징한 jar 밖에 있는 프로파일 전용 애플리케이션 프로퍼티 (`application-{profile}.properties` 또는 YAML로 작성한 파일)

> 같은 위치에 `.properties`와 `.yml` 설정 파일이 둘 다 있다면 `.properties`를 우선시한다.
> 

보통 `src/main/resources` 내에 설정 파일을 위치시키는데 해당 경로는 `classpath`로 설정되어있으므로 3, 4번에 해당하고, `application.properties`를 기준으로 현재 활성화된 프로파일의 속성을 오버라이드 시킨다.

좀 더 우선순위가 낮은 설정파일을 추가하고 싶다면 `resources/.config` 내에 위치시키면 된다.

5~8번의 경우 `classpath` 외부에 위치하는데 애플리케이션을 실행하는 위치에 있는 설정파일이다.

`gradle`이나 `java` 명령어로 스프링 부트 애플리케이션을 실행시킬 때 보통 `.jar`파일이 있는 곳에서 실행하게 된다. 그 위치에서 `config` 디렉토리 하위 위치한 설정 파일이 `classpath`에 있는 설정파일을 오버라이드한다.

그리고 `.jar` 파일과 동일한 디렉토리 내에 있는 설정 파일이 가장 높은 우선순위를 가진다.

따라서 빌드 없이 외부 설정 파일만 바꾼 뒤 애플리케이션을 재시작하면 설정을 바꿔서 실행시킬 수 있다.

특히 로컬에서 IDE 등을 이용하는 경우 프로젝트 루트 디렉토리 또는 루트 디렉토리 내 `config` 디렉토리에 설정 파일을 따로 사용하고 `.gitignore`에 추가하게 되면, 암호 등과 같은 민감한 정보를 `git`에 포함시키지 않고 공유할 수 있다.

예시를 위해 다음과 같이 `name` 프로퍼티를 사용하는 `@Component`를 개발한다고 해보자.

```java
@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

애플리케이션 클래스패스(ex. jar 내부)에 적당한 `name` 프로퍼티의 디폴트 값을 제공하는 `application.properties` 파일이 있을 수 있다. 다른 환경에서 실행할 때는 `name`을 재정의하는 `application.properties` 파일을 jar 외부에서 제공할 수 있다. 일회성 테스트라면 커맨드라인 스위치로 프로퍼티를 특정해서 (ex. `jar -jar app.jar --name="Spring"`) 기동할 수도 있다.

> `env`, `configprops` 엔드포인트는 어떤 프로퍼티에 그 값이 왜 들어가 있는지를 확인할 때 유용하다. 이 두 엔드포인트를 통해 기대와 다른 프로퍼티 값을 진단해볼 수 있다.
> 

## Accessing Command Line Properties

SpringApplication은 기본적으로 모든 커맨드라인 옵션 인자(즉, `--server.port=9000` 같이 `--` 로 시작하는 인자)를 `property` 로 변환해서 스프링 `Environment` 에 추가한다. 앞에서도 언급했지만, 커맨드라인 프로퍼티는 항상 파일 기반 프로퍼티 소스보다 우선한다.

커맨드라인 프로퍼티가 `Environment` 에 추가되는게 싫다면 `SpringApplication.setAddCommandLineProperties(false)` 로 비활성화 할 수 있다.

## External Application Properties

스프링 부트는 어플리케이션을 시작하면서 다음 위치에서 자동으로 `application.properties`와 `application.yaml` 파일을 찾아 로드한다.

1. 클래스패스에서
    1. 클래스패스 루트
    2. 클래스패스 `/config` 패키지
2. 현재 디렉토리에서
    1. 현재 디렉토리
    2. 현재 디렉토리 밑에 있는 `/config` 디렉토리
    3. 하위 `/config` 디렉토리 바로 밑에 있는 디렉토리들

이 항목들은 우선 순위에 따라 정렬된다 (밑에 있는 항목이 위에 있는 항목을 재정의한다). 로드한 파일에 있는 도큐먼트들은 스프링 `Environment`에 `PropertySources`로 추가된다.

설정 파일명으로 `application`이 마음에 들지 않으면 environment 프로퍼티 `spring.config.name`을 지정해서 다른 파일명으로 전환하면 된다. 예시로 `myproject.properties`와 `myproject.yaml` 파일을 찾고 싶다면 다음과 같이 애플리케이션을 실행하면 된다.

```java
$ java -jar myproject.jar --spring.config.name=myproject
```

environment 프로퍼티 `spring.config.location` 을 사용해 참조할 위치를 명시할 수도 있다. 이 프로퍼티는 확인할 위치를 하나 이상, 쉼표로 구분해서 받는다.

다음은 두 개의 다른 파일을 지정하는 예시다.

```java
$ java -jar myproject.jar --spring.config.location=\
		optional:classpath:/default.properties, \
		optional:classpath:/override.properties
```

> location이 없을 수도 있기에 파일이 존재하지 않아도 상관없다면 `optional:` 을 prefix로 사용하라.
> 

> `spring.config.name` , `spring.config.location` , `spring.config.additional-location` 은 로드할 파일을 결정하기 때문에 읽어가는 시점이 훨씬 빠르다. 따라서 이 값들은 environment 프로퍼티 (보통 OS 환경 변수나, 시스템 프로퍼티, 커맨드라인 인자)로 정의해야 한다.
> 

`spring.config.location` 에 디렉토리(파일이 아닌)를 지정한다면 `/` 로 끝나야 한다. 디렉토리엔 런타임에 로드하기 전 `[spring.config.name](http://spring.config.name)` 으로 생성한 이름이 덧붙여진다. `spring.config.location` 에 지정한 파일들은 그래도 임포트한다.

> location 값으로 사용한 디렉토리와 파일은 모두 프로파일 전용 파일도 함께 확인하기 위해 치환(expand)된다. 예를 들어 `classpath:myconfig.properties` 가 `spring.config.location` 으로 있는 경우 적절한 `classpath:myconfig-<profile>.properties` 파일도 로드되는 걸 알 수 있다.
> 

대부분은 단일 파일이나 디렉토리를 참조하는 `spring.config.location` 항목을 추가한다. location은 정의된 순서대로 처리되며, 나중에 정의한 location이 앞에 나온 location을 재정의할 수 있다.

location 설정이 복잡한데 프로파일별 설정 파일까지 사용한다면, 스프링 부트가 이 파일들을 그룹화할 방법을 알 수 있도록 힌트를 더 제공해야 할 수도 있다. location 그룹은 모두 동일한 레벨로 간주하는 location의 모음이다. 예를 들어 모든 클래스패스 location을 그룹으로 묶고, 모든 외부 location은 또 다른 그룹으로 묶고 싶을 수 있다. 하나의 location 그룹 내에 있는 항목은 `;` 로 구분해야 한다.

`spring.config.location` 을 사용해 설정한 location은 디폴트 location을 대체한다. 예를 들어 `spring.config.location` 을 `optional:/custom-config/, optional:file:./custom-config/` 로 설정했다면 고려하는 전체 location 셋은 다음과 같다.

1. `optional:classpath:custom-config/`
2. `optional:file:./custom-config/` 

location을 대체하기 보단 별도 location을 더 추가하고 싶은 거라면

`spring.config.additinal-location` 을 사용하면 된다. 추가된 location에서 로드한 프로퍼티는 디폴트 location에 있는 프로퍼티를 재정의할 수 있다. 예를 들어 `spring.config.additional-location` 을 `optional:classpath:/custom-config/,optional:file:./custom-config/` 로 설정했다면 고려하는 전체 location 셋은 다음과 같다.

1. `optional:classpath:/;optional:classpath:/config/`
2. `optional:file:./;optional:file:./config/;optional:file:./config/*/`
3. `optional:classpath:custom-config/`
4. `optional:file:./custom-config/`

이 검색 순서를 잘 활용하면 설정 파일 하나에 기본값을 지정하고, 다른 설정 파일에선 필요에 따라 이 값을 재정의할 수 있다. 애플리케이션의 기본값은 디폴트 location 중 하나에 있는 `application.properties` (또는 `spring.config.name` 으로 지정한 다른 베이스 이름)에 제공하면 된다. 이 기본값들은 런타임에 커스텀 location 중 하나에 있는 다른 파일로 재정의될 수 있다.

> 시스템 프로퍼티가 아닌 환경 변수를 사용할 때는 대부분의 운영 체제에서 키 이름을 마침표로 구분할 수 없다. 하지만 그대신 밑줄을 사용할 수 있다 (ex. `spring.config.name` 대신 `SPRING_CONFIG_NAME`).
> 

> 애플리케이션을 서블릿 컨테이너나 애플리케이션 서버에서 실행한다면 JNDI 프로퍼티(`java:comp/env`)나 서블릿 컨텍스트 init 파라미터를 환경 변수나 시스템 프로퍼티 대용으로 사용할 수 있다.
> 

## Profile Specific Files

스프링 부트는 `application` 프로퍼티 파일뿐 아니라, `application-{profile}`을 네이밍 컨벤션으로 사용하는 프로파일 전용 파일도 로드해본다. 예를 들어 어플리케이션이 `prod` 라는 프로파일을 활성화하고 YAML 파일을 사용한다면, `application.yaml` 과 `application-prod.yml` 을 둘 다 찾는다.

프로파일 전용 프로퍼티는 표준 `application.properties` 와 동일한 위치에서 로드하며, 프로파일 전용 파일은 항상 프로파일을 특정하지 않은 파일보다 우선시된다. 프로파일을 여러 개 지정했을 때는 마지막 프로파일이 이기는 전략(last-wins strategy)을 적용한다. 예를 들어, `spring.profiles.active` 프로퍼티에 `prod, live` 프로파일을 지정하면 `application-prod.properties` 에 있는 값은 `application-live.properties` 에 있는 값으로 재정의될 수 있다.

마지막 프로파일이 이긴다는 이 전략은 location 그룹 레벨에 적용된다.
`spring.config.location` 값 `classpath:/cfg/, classpath:/ext/` 는 `classpath:/cfg/;classpath:/ext/` 와는 재정의 규칙이 다르다.
예를 들어, 위에서 언급한 `prod,live` 예시를 계속 이어가자면, 다음과 같은 파일이 있을 수 있다.

```java
/cfg
application-live.properties
/ext
application-live.properties
application-prod.properties
```

`spring.config.location` 에 `classpath:/cfg/,classpath:/ext/` 를 사용하면, `/ext` 의 파일보다 먼저 `/cfg` 의 파일들을 전부 처리한다.

1. `/cfg/application-live.properties`
2. `/ext/application-prod.properties`
3. `/ext/application-live.properties`

이 값 대신 `classpath:/cfg/; classpath:/ext/` 를 사용하면 (`;`를 구분자로 사용), `/cfg`와 `/ext`를 동일한 레벨로 처리한다.

1. `/ext/application-prod.properties`
2. `/cfg/application-live.properties`
3. `/ext/application-live.properties`

`Environment`는 활성 프로파일을 설정하지 않았을 때 사용하는 디폴트 프로파일 셋을 가지고 있다 (기본값은 `Default`). 명시적으로 활성화한 프로파일이 없으면 `application-default` 에서 프로퍼티를 조회한다.

> 프로퍼티 파일들은 무조건 한 번만 로드한다. 이미 프로파일 전용 프로퍼티 파일을 직접 임포트했다면, 다시 임포트하지 않는다.
> 

## 참고 자료

[토리맘 - Externalized Configurati](https://godekdls.github.io/Spring%20Boot/externalized-configuration/)

[스프링 부트 설정 파일 우선 순위](https://jaime-note.tistory.com/371)