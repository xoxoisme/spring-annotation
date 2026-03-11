# spring-annotation

스프링 어노테이션에 관해 기록하는 공간입니다.
---------------------------------------------

## Annotation

- 사전적 의미는 **주석**이지만, 자에서는 코드 사이에 특별한 의미, 기능을 수행하도록 하는 기술이다.

### 종류

#### @SpringBootApplication

- Spring Boot를 자동으로 실행시켜주는 어노테이션이다.
- Bean 등록은 두 단계로 진행된다.
  - `@ComponentScan`을 통해 Component들을 Bean으로 등록한다.
  - `@EnableAutoConfiguration`을 통해 미리 정의해둔 자바 설정 파일들을 Bean으로 등록한다.(Bean은 스프링 IoC 컨테이너에 의해 인스턴스화 되어 조립되거나 관리되는 객체)

#### @Configuration

- 스프링 IoC Container에게 해당 클래스가 Bean 구성 클래스임을 알려주는 어노테이션이다.
- `@Bean`을 해당 클래스의 메서드에 적용하면 `@Autowired`로 빈을 부를 수 있다.

#### @EnableAutoConfiguration

- Spring Application Context를 만들 때 자동으로 설정하는 기능을 켠다.
- 만약 `tomcat-embed-core.jar`가 존재하면 톰캣 서버가 setting된다.

#### @ComponentScan
- `@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`이 붙은 빈들을 찾아서 Context에 빈을 등록해 주는 어노테이션이다.
- bean 인스턴스를 생성한다.
- `@Component`는 아래에 포함되지 않는 요소 즉, 유틸이나 외부 API 호출 클래스 같은 곳에 사용된다.
- `@Service`는 의미상 다르게 쓰는 것이고, `@Repository` 같은 경우 DB 예외가 터지면 `DataAccessException`을 던지고, `@Controller`는 `HandlerMapping`에 등록하기에 일부 MVC 기능이 동작하지 않을 수 있고, `@Configuration`은 설정값을 정의한다.

#### @Bean
- 개발자가 직접 제어가 불가능한 외부라이브러리 등을 Bean으로 만들 때 사용되는 어노테이션이다.
```java
@Configuration
public class AppConfig {
    
    @Bean
    public MessageSender messageSender() {
        if (isProduction()) {
            return new SmsSender();
        }
        return new EmailSender();
    }
}
```

- 보통 외부 라이브러리에 많이 사용되고 메서드 기준으로 사용된다.

#### @Autowired
- 필드, setter 메서드, 생성자에 사용하며 타입에 따라 알아서 Bean을 주입해주는 역할을 한다.
- 객체에 대한 **의존성을 주입**시킨다.

##### 주입 방식에 따른 특징
```java
// 필드 주입 방식
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;
}

// setter 주입 방식
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 생성자 주입 방식
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
- 필드 방식은 간단하지만, 테스트가 어렵고 의존성이 숨겨져 있어 요즘에는 권장하지 않는다.
- setter 방식은 선택적 의존성일 경우만 사용한다.
- 생성자 방식은 테스트도 쉽고, 안전해서 대부분 이렇게 사용한다.

#### @Controller
- MVC의 Controller로 사용되고, 클래스 선언을 단순화 시켜준다.

#### @RestController
- Controller 중 View로 응답하지 않는 컨트롤러를 말한다.
- 반환 결과를 JSON으로 반환한다.
- `@ResponseBody` 역할을 자동으로 해준다.

> API + VIEW = `@Controller`
> API = `@RestController`

#### @Service
- 비즈니스 로직을 수행하는 클래스이다.

#### @Repository
- DAO 클래스에 쓰인다.
- DB에 접근하는 클래스이다.

#### @Resource
- Bean 객체를 주입해주는데 `@Autowired`는 타입으로, `@Resource`는 이름으로 연결해준다.

```java
// @Autowired의 경우 NoUniqueBeanDefinitionException 에러 발생
@Repository
public class JpaUserRepository implements UserRepository {
}

@Repository
public class JdbcUserRepository implements UserRepository {
}

// 사용 예제
@Service
public class UserService {

    @Resource(name = "jpaUserRepository")
    private UserRepository userRepository;

}
```

- 하지만, 요즘은 `@Qualifier`나 `Primary`를 사용합니다.

#### @PreConstruct, @PostConstruct
- 의존하는 객체 생성 이후, 초기화 작업을 위해 실행해야할 메서드 앞에 붙인다.

#### @PreDestroy
- 객체를 제거하기 전, 작업을 수행해야 하는 메서드 앞에 붙인다.

#### @PropertySource
- 프로퍼티 파일을 환경변수로 로딩하게 해준다.

#### @Lazy
- **지연 로딩**을 지원한다.
- 실제로 사용될 때 스프링에서 Bean을 생성한다.

#### @Value
- 프로퍼티에서 값을 가져와 적용할 때 사용한다.

#### @RequestMapping
- URL 요청이 들어오면 어떤 메서드가 처리할 지 매핑해주는 어노테이션이다.

#### @CookieValue
- 쿠키 값을 파라미터로 전달 받고, 없으면 `500` 에러를 발생시킨다.

#### @CrossOrigin
- CORS 보안상의 문제로 다른 **origind의 AJAX 요청을 방지하기 위해** 사용한다.

```java
@CrossOrigin(origins = "http://localhost:3000")
@GetMapping("/users")
public List<User> getUsers() {
    return userService.findAll();
}
```
- 개발 환경에서는 보통 전역 설정을 하지만, 운영 환경에서는 외부 파트너 api 호출 같은 경우 자주 사용된다.

#### @ModelAttirbute
- VIEW에서 전달해 준 파라미터를 DTO의 멤버 변수로 매핑해주는 어노테이션이다.
- 태그의 `name` 값과 변수명이 일치해야 한다.

#### @GetMapping
```java
@RequestMapping(value = "/users", method = RequestMethod.GET)
public List<User> getUsers() {
}
```
- 사실 위처럼 사용이 가능하지만, 가독성을 위해 `GET`,`POST`, `PUT`, `DELETE`로 사용한다.

