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

## AutoConfiguration

### 구현방법
1. 의존성 추가

```xml
<dependencies>
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-autoconfigure</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-autoconfigure-processor</artifactId>
         <optional>true</optional>
      </dependency>
</dependencies>

<dependencyManagement>
     <dependencies>
         <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-dependencies</artifactId>
             <version>2.1.6.RELEASE</version>
             <type>pom</type>
             <scope>import</scope>
         </dependency>
     </dependencies>
</dependencyManagement>
```
2. @Configuration 파일 작성
3. src/main/resource/META-INF에 spring.factories 파일 만들기
4. spring.factories 안에 자동 설정 파일 추가
 
```xml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  me.junghoon.JunghoonConfiguration
``` 
5. mvn install
