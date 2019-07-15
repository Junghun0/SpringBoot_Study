# SpringBoot_Study

## @SpringBootApplication

@SpringBootConfiguration + @ComponentScan + @EnableAutoConfiguration

```java
//@SpringBootApplication
@SpringBootConfiguration
@ComponentScan
@EnableAutoConfiguration
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```
- @EnableAutoConfiguration (@SpringBootApplication 안에 숨어 있음)
- 빈은 사실 두 단계로 나눠서 읽힘
	- 1단계: @ComponentScan
	- 2단계: @EnableAutoConfiguration
- @ComponentScan
	- @Component
	- @Configuration @Repository @Service @Controller @RestController
- @EnableAutoConfiguration
	- spring.factories
		- org.springframework.boot.autoconfigure.EnableAutoConfiguration
	- @Configuration
	- @ConditionalOnXxxYyyZzz
