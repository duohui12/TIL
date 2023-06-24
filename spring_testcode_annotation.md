## 테스트 코드 작성할 때 사용한 애노테이션 정리

<br>

### [@SpringBootTest](https://spring.io/guides/gs/testing-web/)

> The `@SpringBootTest` annotation tells Spring Boot to look for a main configuration class (one with `@SpringBootApplication`, for instance) and use that to start a Spring application context. You can run this test in your IDE or on the command line (by running `./mvnw test` or `./gradlew test`), and it should pass. Spring interprets the `@Autowired` annotation, and the controller is injected before the test methods are run.

- `package org.springframework.boot.test.context` (스프링부트 테스트 패키지)

- @SpringBootTest 애노테이션은 스프링부트에게 메인 configuration클래스를 (예를들어 @SpringBootApplication이 붙은 클래스) 찾아서 스프링 애플리케이션 컨텍스트를 시작하도록 지시한다. 이 테스트를 IDE나 명령줄에서 실행할 수 있다 (`./gradlew test` 또는 `./mvnw test`). 스프링은 `@Autowired` 애노테이션을 해석하고, 테스트 메소드가 실행되기 전에 컨트롤러를 주입한다. 

- 예제코드

  ```java
  package com.example.testingweb;
  import static org.assertj.core.api.Assertions.assertThat;
  import org.junit.jupiter.api.Test;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  
  @SpringBootTest
  public class SmokeTest {
  
  	@Autowired
  	private HomeController controller;
  
  	@Test
  	public void contextLoads() throws Exception {
  		assertThat(controller).isNotNull();
  	}
  }
  ```

<br>

<br>

### [@AutoConfigurationMockMvc](https://spring.io/guides/gs/testing-web/)

> Another useful approach is to not start the server at all but to test only the layer below that, where Spring handles the incoming HTTP request and hands it off to your controller. That way, almost of the full stack is used, and your code will be called in exactly the same way as if it were processing a real HTTP request but without the cost of starting the server. To do that, use Spring’s `MockMvc` and ask for that to be injected for you by using the `@AutoConfigureMockMvc` annotation on the test case. 

- `package org.springframework.boot.test.context` (스프링부트 테스트 패키지)

- 서버를 전혀 시작하지 않고도 스프링이 들어오는 HTTP 요청을 처리하고 컨트롤러에게 전달하는 작업의 아래 계층만 테스트하는 방법이 있다. 이 방법은 거의 모든 스택이 사용되고, 서버를 시작하지 않아도 실제 HTTP 요청 방식과 정확히 동일한 방식으로 호출된다. 이를 위해 스프링의 `MockMvc` 를 사용하고, 테스트 케이스에서 `@AutoConfigurationMockMvc` 애노테이션을 사용해서 자동 주입되도록 요청한다. 

- 예제코드

  ```java
  package com.example.testingweb;
  
  //...
  
  @SpringBootTest
  @AutoConfigureMockMvc
  public class TestingWebApplicationTest {
  
  	@Autowired
  	private MockMvc mockMvc;
  
  	@Test
  	public void shouldReturnDefaultMessage() throws Exception {
  		this.mockMvc.perform(get("/")).andDo(print()).andExpect(status().isOk())
  				.andExpect(content().string(containsString("Hello, World")));
  	}
  }
  ```

<br>

<br>

### [@MockBean](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito/MockBean.html)

> Annotation that can be used to add mocks to a Spring [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/6.0.10/javadoc-api/org/springframework/context/ApplicationContext.html). Can be used as a class level annotation or on fields in either `@Configuration` classes, or test classes that are `@RunWith` the [`SpringRunner`](https://docs.spring.io/spring-framework/docs/6.0.10/javadoc-api/org/springframework/test/context/junit4/SpringRunner.html).
>
> Mocks can be registered by type or by [`bean name`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito/MockBean.html#name()). When registered by type, any existing single bean of a matching type (including subclasses) in the context will be replaced by the mock. When registered by name, an existing bean can be specifically targeted for replacement by a mock. In either case, if no existing bean is defined a new one will be added. Dependencies that are known to the application context but are not beans (such as those [`registered directly`](https://docs.spring.io/spring-framework/docs/6.0.10/javadoc-api/org/springframework/beans/factory/config/ConfigurableListableBeanFactory.html#registerResolvableDependency(java.lang.Class,java.lang.Object))) will not be found and a mocked bean will be added to the context alongside the existing dependency.
>
> When `@MockBean` is used on a field, as well as being registered in the application context, the mock will also be injected into the field. 

- `package org.springframework.boot.test.mock.mockito` (스프링부트 테스트 패키지)

- 스프링 ApplicationContext에 mocks(가짜 객체)를 추가하기 위해 사용되는 애노테이션이다. 클래스 레벨 애노테이션으로 사용되거나, `@Configuration` 클래스 또는`@RunWith(SpringRunner.class)` 테스트 클래스에서 필드로 사용된다. 

- mocks(가짜 객체)는 타입 또는 빈 이름으로 등록된다. 타입으로 등록될 때, 컨텍스트에 있는 일치하는 유형(하위 클래스 포함)의 기존 단일 빈은 모두 mock bean으로 대체된다. 만약 그런 빈이 없다면 새로운 mock bean 이 등록된다. 이름으로 등록될 경우, 기존에 존재하는 빈은 mock bean에 의해 교체 대상이 될 수 있다. 두 경우 모두 기존의 빈이 정의되어 있지 않으면 새로운 빈이 추가된다. 

- 빈으로 등록되지 않은 종속성 (직접 등록된 종속성)은 찾을 수 없고, 기존 종속성과 함께 mock bean이 컨텍스트에 추가된다. 
  -  나는 이 부분을 이렇게 해석했다. 예를 들어 TaskController에서 TaskService를 의존한다고 가정해보자. 스프링에 의해 빈으로 등록된 TaskService 인스턴스가 자동 주입되는 방식이 아니라, 컨트롤러 내부에서  `TaskService taskService = new TaskService()` 와 같이 코드를 작성해서 직접 종속성을 등록하는 경우를 의미하는 것 같다. 이럴 경우에는 테스트 클래스에서 `@MockBean private TaskService taskService;` 와 같이 작성해도 TaskController는 기존과 동일하게 컨트롤러 내에서 생성한 taskService 인스턴스를 의존하고, 이 mock bean은 그 인스턴스와는 별개로 컨텍스트에 등록된다. 따라서, 기대했던 (TaskController 내부에 있는 taskService 인스턴스가 mockbean으로 대체되는 것) 테스트 결과를 얻지 못할 수 있다. 

- @MockBean은 필드에서 사용될 때, application context에 등록되는 것과 함께, mock(가짜 객체가)이 필드에 주입된다. 

- 예제코드

  ```java
  @RunWith(SpringRunner.class)
    public class ExampleTests {
   
        @MockBean
        private ExampleService service;
   
        @Autowired
        private UserOfService userOfService;
   
        @Test
        public void testUserOfService() {
            given(this.service.greet()).willReturn("Hello");
            String actual = this.userOfService.makeUse();
            assertEquals("Was: Hello", actual);
        }
   
        @Configuration
        @Import(UserOfService.class) // A @Component injected with ExampleService
        static class Config {
        }
    }
  ```

<br><br>

### [@Nested](https://junit.org/junit5/docs/5.8.1/api/org.junit.jupiter.api/org/junit/jupiter/api/Nested.html)

> `@Nested` is used to signal that the annotated class is a nested, non-static test class (i.e., an *inner class*) that can share setup and state with an instance of its [enclosing class](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Class.html#getEnclosingClass()). The enclosing class may be a top-level test class or another `@Nested` test class, and nesting can be arbitrarily deep.

- `package org.junit.jupiter.api;` (Junit5 부터 나온 기능)

- @Nested는 이 애노테이션이 달린 클래스가 중첩된, non-static 테스트 클래스임을 나타내며, 이 클래스를 감싸고 있는 외부 클래스의 인스턴스와 setup이나 state를 공유할 수 있다는 것을 나타낸다. 외부 클래스는 최상위 테스트 클래스일 수도 있고 또 다른 @Nested 테스트 클래스일수도 있다. 중첩은 임의로 깊게 할 수 있다. 