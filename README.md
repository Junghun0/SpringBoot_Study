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

#### 자동 설정파일

org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
서블릿 웹서버를 자동설정하는 자동설정이다.
자동설정을 통해 톰캣이 만들어진다.

<img width="579" alt="스크린샷 2019-07-16 오후 1 22 12" src="https://user-images.githubusercontent.com/30828236/61266048-ff028400-a7cd-11e9-8b0c-255b7904276d.png">
<img width="762" alt="스크린샷 2019-07-16 오후 1 24 07" src="https://user-images.githubusercontent.com/30828236/61266042-f90ca300-a7cd-11e9-90ab-001f1c550967.png">
<img width="646" alt="스크린샷 2019-07-16 오후 1 31 36" src="https://user-images.githubusercontent.com/30828236/61266081-16da0800-a7ce-11e9-85e9-aa90c493bb19.png">



## 내장 웹 서버 응용

- 다른 서블릿 컨테이너로 변경
- 웹 서버 사용 하지 않기
- 포트
	- server.port
	- 랜덤 포트
	- ApplicationListner< ServletWebServerInitializedEvent >

<img width="567" alt="스크린샷 2019-07-16 오후 1 51 55" src="https://user-images.githubusercontent.com/30828236/61266961-f3648c80-a7d0-11e9-84ff-7f88e2152a2a.png">

spring-boot-starter-web 에서 가져오는 tomcat 의 사용을 안하도록 설정한다.

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
       <exclusion>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-tomcat</artifactId>
       </exclusion>
   </exclusions>
</dependency>
```

jetty 의존성 추가

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

<img width="569" alt="스크린샷 2019-07-16 오후 1 58 40" src="https://user-images.githubusercontent.com/30828236/61267191-d9777980-a7d1-11e9-98f8-01fc61fd7c81.png">

spring-boot-starter 아래에 있는 tomcat 이 사라지고 jetty 의존성이 추가된 것을 확인할 수 있다.

- resources/application.properties
	- 웹 서버 사용하지 않기<br/>
	spring.main.web-application-type=none
	- 랜덤 포트 사용<br/>
	server.port=0
	- ApplicationListner< ServletWebServerInitializedEvent > <br/>
	
```java
	
@Component
public class PortListener implements ApplicationListener<ServletWebServerInitializedEvent> {
    @Override
    public void onApplicationEvent(ServletWebServerInitializedEvent servletWebServerInitializedEvent) {
        ServletWebServerApplicationContext servletWebServerApplicationContext = servletWebServerInitializedEvent.getApplicationContext();
        System.out.println(servletWebServerApplicationContext.getWebServer().getPort());
    }
}
```
