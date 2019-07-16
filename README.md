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


- 덮어쓰기 방지하기
	- @ConditionalOnMissingBean
- 빈 재정의 수고 덜기
	- @ConfigurationProperties(“junghoon”)
	- @EnableConfigurationProperties(JunghoonProperties)
	- 프로퍼티 키값 자동 완성
	
```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-configuration-processor</artifactId>
   <optional>true</optional>
</dependency>
```


## 내장 웹 서버 이해

- 스프링 부트는 서버가 아니다.
	- 톰캣 객체 생성
	- 포트 설정
	- 톰캣에 컨텍스트 추가
	- 서블릿 만들기
	- 톰캣에 서블릿 추가
	- 컨텍스트에 서블릿 맵핑
	- 톰캣 실행 및 대기

```java
public class DemoApplication {

    public static void main(String[] args) throws LifecycleException {
        //SpringApplication.run(DemoApplication.class, args);

        Tomcat tomcat = new Tomcat();
        tomcat.setPort(8080);

        Context context = tomcat.addContext("/", "/");

        HttpServlet servlet = new HttpServlet() {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                PrintWriter writer = resp.getWriter();
                writer.println("<html><head><title>");
                writer.println("Hey, Tomcat");
                writer.println("</title></head>");
                writer.println("<body><h1>Hello Tomcat</h1></body>");
                writer.println("</html>");
            }
        };

        String servletName = "helloServlet";
        tomcat.addServlet("/", servletName, servlet);
        context.addServletMappingDecoded("/hello", servletName);

        tomcat.start();
        tomcat.getServer().await();
    }
}
```
- 이 모든 과정을 보다 상세히 또 유연하고 설정하고 실행해주는게 바로 스프링 부트의 자동 설정.
	- ServletWebServerFactoryAutoConfiguration (서블릿 웹 서버 생성)
		- TomcatServletWebServerFactoryCustomizer (서버 커스터마이징)
- DispatcherServletAutoConfiguration
	- 서블릿 만들고 등록
