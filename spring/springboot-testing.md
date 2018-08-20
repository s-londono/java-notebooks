# Unit Testing with Spring Boot

* Include the dependency spring-boot-starter-test with test scope, so that it’s not put into releases:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

## Simple Tests

* The simplest of tests requires no annotations whatsoever on the class. Only on the test methods:

```java
public class TestSerialization {
  private static Logger logger = LoggerFactory.getLogger(TestSerialization.class);
  
  ObjectMapper jsonMapper = new ObjectMapper();

  @Test
  public void testFallbackDeserialization() {  
    String strFllbk = "{\"object\":\"page\",\"entry\":[]}";
    
    GetCallback cllbk = null;
    
    try {
      cllbk = jsonMapper.readValue(strFllbk, GetCallback.class);
    } 
    catch (IOException e) {
      e.printStackTrace();
    }
    
    Assert.assertNotNull(cllbk);
  }

  ...
}
```

## Advanced Tests

* Tests should be set to run with SpringRunner.class, otherwise annotations such as @SpringBootTest will be ignored. 

* Spring Boot tests are annotated with @SpringBootTest, which replaces the @ContextConfiguration annotation used in the Standard Spring Test Framework. 

* By default, @SpringBootTest looks for a SpringAplication class in the project (i.e. a @SpringBootConfiguration) and uses it to load the ApplicationContext. This is the case when no @ContextConfiguration is specified and no @Configuration classes are nested in the test. 

   Note that @ActiveProfiles can be used to define the profile to run the test with and the  properties attribute of @SpringBootTest can override property values.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(properties={"cat.prop1=[\"v1\"]", "cat.prop2=abc"})
@ActiveProfiles({"local"})
public class SpringBootAutoLoadedTest {
    ...
}
```

## Customizing and extending test ApplicationContext

* To define use a custom Application Context configuration from scratch, add a nested @Configuration class to the test. Spring Boot Test will use this @Configuration class to load the ApplicationContext instead of the project's SpringApplication. That is, a nested @Configuration *overrides* the default Application Context configuration of Spring Boot Tests.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootManuallyLoadedTest {
  ...

  @Configuration
  public static class ManualTestConfig {
    // Bean definitions...
  }

}
```

* To combine the default test ApplicationContext configuration with some customizations, use @TestConfiguration. In contrast with a nested @Configuration, a nested @TestConfiguration does not prevent the SpringApplication from being used to load the Application Context of the test, but is added up to its configuration. That is, a nested @Configuration *extends* the default Application Context configuration of Spring Boot Tests.

   According to the documentation, @TestConfiguration:
   > "Can be used to define additional beans or customizations for a test. Unlike regular @Configuration classes the use of @TestConfiguration does not prevent auto-detection of @SpringBootConfiguration".

```java
@RunWith(SpringRunner.class)
@SpringBootTest(properties={"cat.prop1=abc"})
@ActiveProfiles({"local"})
public class SpringBootAutoLoadedTest {
  ...

  @TestConfiguration
  public static class BotApiBeansConfig {
    
    @NotBlank
    @Value("${cat.prop1}")
    private String prop1;

    @Bean("someBeanName")
    SomeBean someBean() {
      return new SomeBean();
    }

  }

}
```

* It's also possible to specify an @ContextConfiguration on the test. In that case, the @SpringBootTest will not use the SpringApplication configuration to load the ApplicationContext.  This is equivalent to setting the classes attribute in @SpringBootTest to a class annotated with @Configuration (e.g. @SpringBootTest(classes={BotCoreService.class}))
<!-- TODO: Validate this claim. See example at bot-core.BotPingCommandTest -->


## References

https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html

https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/testing.html#testing
