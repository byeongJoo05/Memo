# Config

## Spring Cloud Config

- Spring Cloud Config는 설정파일을 외부(git 등)에서 관리하고 각 프로그램은 Spring Cloud Config Client(이하 Client)를 라이브러리에 포함시킨 뒤 Spring Cloud Config Server(이하 Server) 연결 정보를 기술하면 프로그램 시작 시 Spring Boot Application Properties 정보를 자동으로 읽어올 수 있다. 뿐만 아니라, `@RefreshScope`를 이용하면 변경된 설정 정보를 프로그램 재시작 없이 자동 반영할 수도 있다.

### 왜 Spring Cloud Config를 사용해야 될까?

특정 확률에 따라 광고를 내보내는 기능을 개발한다고 해보자. 그리고 확률 값이 변경될 때마다 수익이 어떻게 변하는지 확인해보는 A/B 테스트를 진행할 것이다. 처음엔 5%로 설정되었다가 테스트가 끝난 후엔 다시 10%로 바꿔야 한다. 확률 값을 프로젝트 설정 파일에서 읽어오거나 특정 변수에 대입하여 사용할 텐데 이런 경우 **값을 변경할 때마다 다시 빌드 후 배포하는 과정이 필요**하게 된다.

이럴 때 **Spring Cloud Config**가 도움을 줄 수 있다. 분산된 환경에서 **설정 파일을 외부로 분리할 수 있도록** 해준다. 개발/테스트 환경 그리고 운영 환경에서까지 모든 환경 구성을 간편하게 관리할 수 있다. 설정을 위한 별도의 서버를 구성하기 때문에 실행 중인 애플리케이션이 서버에서 설정 정보를 받아와 갱신하는 방식이다. 즉 실행 중에 설정값 변경이 필요해지면, 설정 서버만 변경하고 애플리케이션은 갱신하도록 해주기만 하면 된다. 따라서 **설정이 바뀔 때마다 빌드와 배포가 필요 없는 구조**이다.

## Spring Cloud Config의 기본 구조

![2020-01-30-introduction-to-spring-cloud-config-1](https://github.com/byeongJoo05/Memo/assets/84984586/bd58cada-21df-407d-a3ca-9697ffbbddb9)

- 먼저 클라이언트는 서버로 **설정값을 요청**하면 서버는 설정 파일이 위치한 **Git 저장소에 접근**한다. 서버는 Git 저장소로부터 최신 설정값을 받고 클라이언트는 다시 서버를 통해 **최신 설정값**을 받는다. 만일, **사용자가 설정 파일을 변경하고 Git 저장소를 업데이트했다면 애플리케이션(클라이언트)의 `actuator/refresh` 엔드 포인트를 호출하여 설정값을 변경**하도록 한다. 여기서의 설정값 갱신을 위한 엔드 포인트 호출은 각각의 클라이언트를 호출해야 하며 호출된 클라이언트만 반영된다.

## Spring Cloud Config Server

- Server에서 git 등에 정의된 설정 정보를 읽어와 Client에 제공할 수 있다. 이 때 다수의 설정 정보를 체계적으로 제공하기 위해 URI의 형태에 따라 설정 파일을 구분할 수 있다. 가장 쉬운 방법은 Client의 `spring.application.name`와 `spring.profiles.active`를 이용해서 읽어올 설정 파일을 URI로 구분하는 방법이다.

- Server에서 설정해줘야 하는 기본적 설정은 Client가 설정 정보를 받아가기 위해 기동되어야 하는 WAS의 포트인 `server.port`와 설정 정보가 저장되어 있는 문서의 위치를 지정하는 `spring.cloud.config.server` 이하의 저장소 정보(git, svn 등) 이다. 추가적으로 Server에서는 URI를 이용해 설정 정보를 제공하기 때문에 다른 웹프로그램과 동시에 사용하게 되면 URI를 한 쪽이 선점하게 되어 문제가 발생할 수 있다. 그래서 `spring.cloud.config.server.prefix` 설정을 이용해 특정 URI(예를 들어 /config와 같은) 이하에서만 동작할 수 있도록 제한을 할 수 있다.

- 가장 많이 이용하는 방식은 git 저장소에 설정 정보를 저장해두는 것인데, 이 때 인증을 위해 username과 password 를 설정에 넣어 사용하는 방식을 제공하지만 개인키/공개키를 이용해 가져올 수도 있다. 개인의 인증정보가 소스 저장소에 저장되어 돌아다니는 것은 좋지 않기에 개인키/공개키를 이용해 정보를 가져올 수 있도록 구성하는 것이 좋다.

```yaml
server:
  port: 9090

spring:
  cloud:
    config:
      server:
        uri: config server의 git 주소
        search-paths: "*, {application}"
        username: username
        password: password
      prefix: /config # Eureka와 같이 사용하기 위해 선언
eureka:
  client:
    register-with-eureka: false
```

위 코드 스니펫은 기본 설정을 적용한 형태이다. 위에 설명되었듯 하나의 서버에서 여러 서비스를 제공하기 위해 `prefix`를 지정하였다. 그래서 또다른 서비스인 Eureka와 동시 사용이 가능하다.

그리고 저장소에서는 아래와 같이 설정 파일을 등록해서 Client가 사용할 수 있게 한다. 아래 이미지에서는 ribbon에서 사용할 설정 정보를 Spring Cloud Config를 이용해 가져올 수 있도록 설정한다.

![Untitled](https://github.com/byeongJoo05/Memo/assets/84984586/f06e56c7-644a-4576-b95d-90a6d404807e)

## Spring Cloud Config Client

- Client는 Server에서 설정 정보를 가져오는 설정을 통해 외부에서 설정을 관리할 수 있도록 기능을 구성할 수 있다. 이 때, application properties에 Server 연결 정보를 기술할 경우 Spring의 실행 순서에 의해 미리 설정 정보를 가져올 수 없기에 bootstrap properties 파일을 별도로 생성하여 먼저 연결할 수 있도록 해야 한다.

- 가장 중요한 설정 정보는 `spring.application.name`, `spring.profiles.active` 와 `spring.cloud.config.uri` 이다.
uri를 이용해 Server에 접속한 뒤 application name과 profile name을 이용해 설정 정보를 가져올 수 있다. Spring Boot 에서는 보통 내장 WAS를 이용해 자체 실행하기 때문에 spring.profiles.active는 실행 시 -D 옵션으로 제공되는 경우가 많아 bootstrap properties 파일에서는 생략될 수 있다.

- 아래 이미지에서는 기본적 bootstrap properties를 설정한 모습을 보여준다. 위의 Server 설정에서 `prefix`를 `/config` 으로 설정하였기에 uri에 /config이 추가되어 있다.

```yaml
spring:
  application:
    name: controller-test
  cloud:
    config:
      uri: <host>/config
```

### 애플리케이션 실행(구동) 시 처음 INFO로 나오는 Properties Mapping

- ************************************************************************************************************Fetching config from server at : http://localhost:8888************************************************************************************************************
    - 애플리케이션이 시작되며 Config Server로부터 설정 내용을 가져온다. 그리고 애플리케이션이 정상적으로 시작되었다면, 아래의 내용을 확인할 수 있을 것이다.
- **Tomcat started on port(s): 9999 (http)**
    - 설정 파일의 내용을 스프링 애플리케이션이 참조하여 해당 애플리케이션의 포트를 9999로 설정한 것이다.
    - 스프링 부트의 특징 중 하나는 각종 설정의 오버라이드가 매우 쉽다는 점인데, 이것들은 서로 참조되는 순서가 있다.
        - 첫번째로 참조되는 것이 바로 bootstart.properties로, 이 파일에 있는 내용은 애플리케이션의 이름, config server의 위치와 같이 거의 변경되지 않는 내용을 넣는다.
        - 두번째로 참조되는 것이 이 경우에는 Config Server에 있는 설정 파일의 내용인데, 여기서 역시 참조 순서가 있다. Config Server가 참조하는 코드 저장소에 있는 설정 파일의 이름에 application.properties가 있다면, 여기에 있는 내용은 모든 Config Server를 참조하는 Config Client들이 참조하는 전역 설정과 같은 것이다. 그리고 이와 함께 `[application-name].properties`가 있다면, 이 **application.properties 위에 설정을 merge 해서 사용**한다. 이때 **동일한 설정이 있다면 이기는 쪽은 애플리케이션이름.프로퍼티 파일이 되겠다.**
    - 이러한 우선 순위 규칙에 따라 애플리케이션 설정의 참조 순서를 적용할 수 있으며, 더욱 중요한 것은 이를 통해 설정과 코드를 분리할 수 있다는 것이다. 스프링부트 어플리케이션은 시작할 때 서버의 설정 내용을 참조하여 스스로 설정하며, 이는 12 factor 애플리케이션의 config 부분에 취급되는 매우 중요한 내용 중 하나가 되겠다.
- 그리고 각 설정 파일을 환경 별로 생성할 필요 없이 사용될 환경을 지정해서 사용하는 것도 가능하다. 이는 `Profile` 이라고 불리는데, 위의 어플리케이션의 시작 로그를 살펴 보면 아래와 같은 로그가 남는 것을 확인할 수 있다.
    - **************************No active profile set, falling back to default profiles: default**************************
    - 따라서 사용하는 환경 별로 test, staging, prd와 같은 형태로 구성하여 사용할 수 있다.

**예시**

- `bootstrap.yml`에 실행할 `profile` 옵션을 넣었기에 아무런 설정없이 실행해도 된다. 실행하면서 직접 환경에 지정하고 싶은 경우는 `-Dspring.profiles.active=dev` 처럼 옵션을 넣어주면 된다.
- Client가 구동될 때 시작 로그가 찍힌다면 다음과 같은 형태로 나오게 된다.

```java
Fetching config from server at : http://localhost:8088
Located environment: name=config, profiles=[dev], label=null, version=760804e4ac41eee7a8d7cb14f42588b3ad7252fc, state=null
Located property source: [BootstrapPropertySource {name='bootstrapProperties-configClient'},
    BootstrapPropertySource {name='bootstrapProperties-https://github.com/madplay/spring-cloud-config-repository/config-dev.yml'}]
```

## Refresh Config

- 위에서 `@RefreshScope`를 이용하여 설정을 프로그램 재기동 없이 반영할 수 있다고 한다. `@RefreshScope`를 Class에 지정하고 `@Value`를 이용해 설정을 읽어오도록 하면 git 등에 저장해둔 값이 변경되었을 때 자동으로 반영할 수 있다.
- 하지만, 저장소에서 변경하는 즉시, 혹은 일정 시간이 지난 뒤 자동으로 반영되는 것은 아니고, Client에 설정이 변경되었으니 다시 읽어오라고 요청해야 반영이 된다. 최신버전의 Spring Boot에서는 이러한 기능이 **spring actuator 에 포함**되었으며 Spring Boot 에서는 `spring-boot-starter-actuator` 라이브러리를 추가하면 쉽게 등록할 수 있다.
- application properties에 다음과 같이 `management.endpoints.web.exposure.include` 값에 `refresh`를 추가한 뒤 웹브라우저에서 `/actuator/refresh` 경로를 호출하면 된다. (구버전에서는 actuator 없이 /refresh를 직접 호출하도록 되어 있다.)

## Spring Cloud Bus

- Spring Cloud Bus (이하 Bus)는 Client의 정보 갱신시 모든 client에 URL를 호출하여 갱신시켜주는 불편함을 줄이기 위해 AMPQ를 이용해 Server에 갱신하라는 지시를 보내주는 라이브러리이다. RabbitMQ와 Kafka를 기본적으로 지원한다.
- 하지만, RabbitMQ와 Kafka는 규모가 큰 시스템이기에 Spring Cloud Config 만을 위해 도입하기에 좋은 판단이 아니다. 최소한 Client의 수가 매우 많아져 일일이 URL을 호출하거나 Jenkins와 같은 Tool로 호출할 때 일부 장비의 장애를 감지하기 어려운 정도의 수가 운영될 때 도입을 검토하는 것이 좋다.

## Spring Cloud Ribbon

- Spring Cloud Ribbon (이하 Ribbon)은 서버의 경로를 별도로 관리하고 Alias로 처리하여 서버의 IP/URL 정보의 변경에 자유롭고 Ribbon 자체적으로 하나의 Alias에 여러 서버 정보를 등록하여 부하를 분산할 수 있으며, 장애가 발생한 장비를 감지하여 이를 자동으로 제외시킬 수도 있는 기능을 제공한다.

## @RibbonClient

- Spring에서 cURL 호출을 하기 위해 RestTemplate을 많이 이용한다. Ribbon을 이용해 서버 목록을 프로그램이 관리하려면 `@RibbonClient`를 이용하여 RestTemplate를 확장시킬 수 있다.

```java
@RibbonClient(name = "service-test")
@Configuration
public class ServiceTestRestTemplateConfig {
    
    @LoadBalanced
    @Bean
    public RestTemplate serviceTestRestTemplate() {
        return new RestTemplate();
    }
}
```

- 위와 같이 설정한 RestTemplate를 다음 이미지와 같이 사용할 수 있다.

```java
@RequiredArgsConstructor
@RestController
public class TestController {

    private final RestTemplate serviceTestRestTemplate;

    @GetMapping("/test2")
    public Mono<Map<String, Object>> test2() {
        ResponseEntity<Map> responseEntity = serviceTestRestTemplate.getForEntity("http://service-test/test2", Map.class);

        return Mono.just(Map.of("status", 0, "message", "success", "test", responseEntity.getBody()));
    }
}
```

- 여기서 service-test라는 경로는 DNS에 등록된 정보가 아니라 @RibbonClient에 name으로 선언된 이름이다.

- 이 service-test는 어떤 주소를 가리키는 걸까? → 이 정보는 Spring Boot에 application.yaml 에서 쉽게 선언할 수 있도록 해두었다.

```yaml
service-test:
  ribbon:
    eureka:
      enabled: false
    listOfServers: localhost:8082, localhost:8083
    ServerListRefreshInterval: 15000
```

`(RibbonClient Name).ribbon.listOfServers`에 접속할 서버 URL을 선언해두면 Ribbon에서 자동으로 목록 중 하나를 선택하여 (이것도 순차적으로 가져올 것인지 랜덤하게 가져올 것인지 설정 가능) name에 치환하여 요청을 보낸다.

## Spring Cloud Config 와 연동

- 여러 WAS가 같은 소스로 배포되어 있을 경우 cURL로 호출하는 URL이 변경되어야 할 경우 혹은 서버가 줄어들거나 늘어나야 할 경우 기존 소스를 수정하고 새로 배포하거나 L4/L7을 이용해 이를 관리해야 한다. 하지만, Ribbon과 Spring Cloud Config를 이용하면 git과 같은 저장소에서 설정을 바꾼 뒤 설정을 다시 읽게만 하면 서비스 중지나 재배포, L4/L7 재시작 없이도 간단하게 적용이 가능하다.

## @RibbonClients

- `@RibbonClient`를 이용해 접속할 Server 정보를 Ribbon에서 관리할 수 있지만, 복수의 Server가 존재할 경우 RestTemplate를 주입(injection)받는 Class를 서버 갯수만큼 생성해야 하는 문제가 발생할 수 있다. 그렇기에 하나의 Class에서 여러 서버 설정을 하기 위해 `@RibbonClient`를 복수를 선언해 사용할 수 있는 `@RibbonClients`를 사용하는 것이 편리하다.

- 아래 코드 스니펫과 같이 하나의 RestTemplate 객체에 복수의 @RibbonClient를 등록하면 restTemplate 으로 주입받아 사용할 때 service-test와 service-was-test 모두 사용할 수 있다.

```java
@RibbonClients(
        value = {
                @RibbonClient(name = "service-test"),
                @RibbonClient(name = "service-was-test")
        }
)
@Configuration
public class ServiceTestRestTemplateConfig {

    @LoadBalanced
    @Bean
    public RestTemplate serviceTestRestTemplate() {
        return new RestTemplate();
    }
}
```

## Spring Cloud Eureka

- **Spring Cloud Eureka (이하 Eureka)는 Service Discovery의 일종**이다. Ribbon과 함께 소개되는 경우가 많은데, **Ribbon은 요청을 받는 서버의 정보를 요청할 곳에서 목록화해서 관리하고 있는 형태라서 요청할 서버에만 설정을 해두면 되는 간단한 형식**이라면, **Eureka는 Eureka Server가 요청을 받는 서버의 Eureka Client가 보내주는 정보를 취합하여 서버로 등록해두고 요청할 서버가 요청을 받는 서버의 정보를 요구하면 이를 제공하는 형태**로 구성되어 있다.

## Eureka Server

- Eureka Server는 `@EnableEurekaServer`를 이용하여 간단하게 설정할 수 있다. Spring Boot에 Eureka Server를 활성화하면 다음과 같이 상태를 조회할 수 있다.

![Untitled 1](https://github.com/byeongJoo05/Memo/assets/84984586/de594766-eb45-462d-9b39-c6ea213afb26)

- 기본적으로 Eureka Server는 ROOT(/) 경로에 위와 같이 모니터링을 할 수 있는 UI를 제공한다. 그리고 `/eureka/**` 경로에 필요한 Endpoint를 제공한다.

## Eureka Client

- Eureka Client를 사용하기 위해서는 @EnableEurekaServer를 선언하면 간단하게 사용할 수 있다. 다음 이미지와 같이 application properties 에 Eureka Server의 service url을 등록하면 Server에 등록된다.

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:9090/eureka/
```

- 프로그램을 실행하면 다음과 같이 모니터링 페이지에 Client 가 등록되어 있는 것을 알 수 있다.

![Untitled 2](https://github.com/byeongJoo05/Memo/assets/84984586/da108ea7-789d-4b62-b34f-5adaabfe08af)

## with Ribbon

- Eureka Client가 되어 Eureka Server에 등록된 프로그램들은 자신의 정보가 Eureka Client에 제공되어 상호 호출이 가능하다. 이 때 Ribbon을 이용해서 호출을 할 수 있다.

```yaml
spring:
  application:
    name: controller-test

service-test:
  ribbon:
    eureka:
      enabled: true
#      enabled: false
#    listOfServers: localhost:8082, localhost:8083
    ServerListRefreshInterval: 15000

service-was-test:
  ribbon:
    eureka:
      enabled: true
    ServerListRefreshInterval: 15000
eureka:
  client:
    service-url:
      defaultZone: http://localhost:9090/eureka/
```

- Ribbon 단독으로 사용할 경우 `listOfServers`를 이용해 RestTemplate으로 접속할 서버의 정보를 나열했다면, Eureka와 연동을 하게 되면 `(server name).ribbon.eureka.enabled`를 true으로 설정하는 것만으로 Eureka Server에서 해당 server name의 접속 정보를 획득하여 이를 이용할 수 있다. 이렇게 Eureka를 연동하게 되면 잦은 서버의 추가/제거 상태에서 설정 파일을 계속 갱신할 필요가 없기 때문에 효율적 서버 관리가 가능하다.

## 참고자료

[MSA 구성을 위한 기술](https://zepinos.tistory.com/82)