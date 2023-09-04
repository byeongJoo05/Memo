# profiles 설정

## yaml 파일 설정하기

- resource 하위 폴더에 작성할 yml 파일은 다음과 같다.
    
    ![img1 daumcdn](https://github.com/byeongJoo05/Memo/assets/84984586/d283d995-0044-465e-bf40-14198df78916)
    

### application.yaml

```yaml
spring:
  profiles:
    active:
      - local
    group:
      local:
        - site-local
        - db-local
      dev:
        - site-dev
        - db-dev
    include:
      - db
      - my-service
      - site
```

- 애플리케이션이 실행될 때 기본적으로 참조하는 파일로서 각 설정들은 다음과 같다.
    - `spring-profiles-active` - **기본적으로 활성화할 profile을 local로 설정**한다.
    - `spring-profiles-group` - **profile group**을 정의할 수 있다.
        - `local` - profile이 local일 경우 **site-local, db-local** 의 profile과 그룹을 정의한다.
        - `dev` - profile이 local일 경우 **site-dev, db-dev** 의 profile과 그룹을 정의한다.
    - `spring-profiles-include` 설정을 통해 어플리케이션을 실행할 때 profile을 포함하여 실행할 수 있다.
        - 즉 **application-db.yml, application-my-site.yml, application-site.yml** 파일을 함께 profile에 포함하여 어플리케이션이 구동된다.

`profile = local`

![img1 daumcdn 1](https://github.com/byeongJoo05/Memo/assets/84984586/b375ca20-2c11-4195-9c9c-5babc7d7f3c0)

`profile = dev`

![img1 daumcdn 2](https://github.com/byeongJoo05/Memo/assets/84984586/cef7b46d-1ce2-4f2b-98c4-840b328b2592)

`profile = prod`

![img1 daumcdn 3](https://github.com/byeongJoo05/Memo/assets/84984586/b0bdd43a-4136-4238-bceb-582cc8b4ba57)

참고로, profile 값 변경은 IntelliJ 기준으로 아래와 같이 설정할 수 있다.

![img1 daumcdn 4](https://github.com/byeongJoo05/Memo/assets/84984586/0a0b6b55-724e-41c0-ab6d-9a93eeb5c334)

**Run > Edit Configurations… > Active profiles**

### application-db.yml

```yaml
#default 공통설정
spring:
  jpa:
    show-sql: false
    open-in-view: false
    database-platform: MYSQL
    hibernate:
      ddl-auto: none
      use-new-id-generator-mappings: true
    properties:
      hibernate.format_sql: true
      hibernate.show_sql: false
      hibernate.dialect: org.hibernate.dialect.MySQL57Dialect
      hibernate.default_fetch_size: ${chunkSize:100}
      hibernate.connection.provider_disables_autocommit: true
      hibernate.jdbc.batch_size: ${chunkSize:100}
      hibernate.order_inserts: true
      hibernate.order_updates: true

--- # local 설정
spring:
  config:
    activate:
      on-profile: "db-local"

  jpa:
    show-sql: true
    database-platform: H2
    hibernate:
      ddl-auto: create-drop

  datasource:
    hikari:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem://localhost/~/divelog;MODE=MySQL;DATABASE_TO_LOWER=TRUE;CASE_INSENSITIVE_IDENTIFIERS=TRUE;
      username: sa
      password:

--- # dev 설정
spring:
  config:
    activate:
      on-profile: "db-dev"

  jpa:
    hibernate:
      ddl-auto: update
  datasource:
    hikari:
      driver-class-name: org.mariadb.jdbc.Driver
      ...
```

- `.yml` 파일은 위와 같이 `---`를 통해 파일 분할이 가능하다. 이로 인해 여러 환경에서 사용될 값을 하나의 파일에 작성할 수 있다.
- `#default`: 프로젝트에 공통 값을 설정한다.
- `spring-config-active-on-profile`: application.yml 파일에서 작성한 group의 profile 이다.
    - 위에서 local, dev일 때 매핑되는 db-local, db-dev 이랑 동일한 네이밍으로 설정한다.
        - 참고로 Spring Boot 2.4 버전부터 설정 파일의 방법이 아래와 같이 변경되었다.
        
        ```yaml
        # before
        spring:
          profiles: "prod"
        secret: "production-password"
        
        # after
        spring:
          config:
            activate:
              on-profile: "prod"
        secret: "production-password"
        ```
        

### application-site.yml

```yaml
site:
  author-name: JuHyun
  author-email: a@a.com

--- # local 설정
spring:
  config:
    activate:
      on-profile: "site-local"

site:
  author-name: JuHyun-local
  author-email: a.local@a.com

--- # dev 설정
spring:
  config:
    activate:
      on-profile: "site-dev"

site:
  author-name: JuHyun-dev
  author-email: a.dev@a.com
```

- `spring-config-active-on-profile`: db와 마찬가지로 application.yml 파일에서 작성한 group의 profile이다.

## Binding

- yml 파일에 설정한 값들을 코드에 바인딩하는 방법이다.

![img1 daumcdn 5](https://github.com/byeongJoo05/Memo/assets/84984586/24f02306-ad46-42f5-997e-922dd73ef182)

- `@Value`를 통해 바인딩하는 방법도 존재하지만, 이번엔 SpringBoot 2.2 버전부터 가능한 `@ConstructorBinding`을 통해 Immutable Properties 설정으로 해보겠다.

### AppConfiguration

```java
@Configuration
@EnableConfigurationProperties({SiteProperties.class})
public class AppConfiguration {
 
}
```

- 바인딩 설정을 위해 AppConfiguration 이라는 클래스 파일을 하나 생성한다.
- 그리고 이 곳에 `@Configuration`, `@EnableConfigurationProperties`를 통해 설정을 한다.
- 이러한 방법 외에도 main 메서드가 존재하는 스프링부트 어플리케이션의 클래스에 `@EnableConfigurationProperties`를 설정해주어도 동일하게 작동한다.

### SiteProperties

```java
@Getter
@ToString
@ConfigurationProperties(prefix = "site")
public class SiteProperties {
 
    private final String authorName;
    private final String authorEmail;
 
    @ConstructorBinding
    public SiteProperties(String authorName, String authorEmail) {
        this.authorName = authorName;
        this.authorEmail = authorEmail;
    }
}
```

- 실제 .yml의 설정 값이 바인딩될 클래스이다.
- `@EnableConfigurationProperties`와 `@ConstructorBinding`을 추가하여 바인딩한다.

![img1 daumcdn 6](https://github.com/byeongJoo05/Memo/assets/84984586/c16019a5-0a30-4d7e-a37a-64eb9002053e)

- `@EnableConfigurationProperties`의 코드를 살펴보면 주석에 setter를 사용하거나 `@ConstructorBinding`을 통해 바인딩이 가능하다고 설명되어 있다.
- Setter의 경우 값변경이 가능하여 Immutable한 상태가 아니므로 `@ConstructorBinding`을 통해 바인딩을 해주었다.

## Binding 테스트하기

- application-site.yml 파일에 작성한 값들에 대해 local, dev 설정을 통해 바인딩이 잘 되었는지 테스트해보자.

### SitePropertiesTest

```java
@SpringBootTest
class SitePropertiesTest {
 
    @Test
    void test(@Autowired SiteProperties properties) {
        assertThat(properties.getAuthorName()).isEqualTo("JuHyun-local");
        assertThat(properties.getAuthorEmail()).isEqualTo("a.local@a.com");
    }
 
}
```

![img1 daumcdn 7](https://github.com/byeongJoo05/Memo/assets/84984586/1b5873c3-73ea-4e69-a03f-651293042c70)

### SitePropertiesDevTest

```java
@ActiveProfiles("dev")
@SpringBootTest
class SitePropertiesDevTest {
 
    @Test
    void test(@Autowired SiteProperties properties) {
        assertThat(properties.getAuthorName()).isEqualTo("JuHyun-dev");
        assertThat(properties.getAuthorEmail()).isEqualTo("a.dev@a.com");
    }
 
}
```

![img1 daumcdn 8](https://github.com/byeongJoo05/Memo/assets/84984586/0f58dda0-b2cd-4c84-ba3f-3b0b2d232d3f)

- application.yml 파일에서 기본 profile을 local로 설정했기 때문에 dev profile을 테스트하기 위해 `@ActiveProfiles("dev")`를 통해 명시적으로 선언하여 테스트한다.

## 참고 자료

[Spring Boot profiles 설정하기](https://zzang9ha.tistory.com/415)