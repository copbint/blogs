[toc]



从原生tomcat到spring mvc，再到spring-boot mvc，tomcat被完全封装起来。

新手如果直接使用spring-boot，甚至都不知tomcat为何物，甚至以为自己没有使用tomcat。

所以从头简单学习一下，感受一下技术的演进过程。



# 基础

## 什么是web服务器

简单来说就是完整的实现http协议的服务端程序：

- 识别正确和错误的HTTP请求；
- 识别正确和错误的HTTP头；
- 复用TCP连接；
- 复用线程；
- IO异常处理；
- ...



## 什么是Servlet

Server Applet（运行在服务端的小程序）

JavaEE定义的一套规范，Servlet体现为一个接口，业务程序员只需要在这个接口中实现业务逻辑即可。

满足Servlet规范的web服务器就能够调用Servlet，执行业务逻辑。

 **Servlet容器**就是存放着Servlet对象的地方。



### **Servlet接口**

Servlet接口中javax.servlet:javax.servlet-api的jar包中定义。

在最基本的Servlet类中，需要实现Servlet接口定义的init()、servic(）、destroy()、getServletConfig()和geServletInfo()方法，其中业务逻辑在service中编写。

### **Servlet接口的演进**

#### 演进1：GenericServlet抽象类

实现Servlet接口的时候必须将所有的方法实现，即使方法中没有任何代码。在GenericServlet抽象类的帮助下，只需要重写service方法即可。

#### 演进2：HttpServlet抽象类

HttpServlet覆盖了GenericServlet类，将ServletRequest和ServletResponse对象分别封装为HttpServletRequest和HttpServletResponse对象。HttpServlet同时实现了service方法，在请求进来时，Web容器首先调用HttpServlet的service方法，并根据请求的类型调用doGet或doPost方法，搜易我们只需要覆盖doGet()和goPost()方法即可。

> [servlet的本质是什么，它是如何工作的？ - bravo1988的回答 - 知乎](https://www.zhihu.com/question/21416727/answer/690289895)
>
> [Servlet 到 Spring MVC 的简化之路](https://juejin.cn/post/6844903570681135117) 



# tomcat

tomcat既是一个web服务器，也是一个Servlet容器。

## 运行servlet

参考文章：

<https://www.liaoxuefeng.com/wiki/1252599548343744/1304265949708322>

写一个简单的Servlet程序，打成war包。

然后下载tomcat，将war包扔到webapps目录下，执行.\startup.bat把tomcat跑起来。就可以访问对应的URL了。

webapps目录下可以多个war包。访问各个war包里面的servlet需要在url前面加上war包名作为前缀。

@WebServlet(urlPatterns = "/")这个注解非常关键，让tomcat可以识别出来这是一个Servlet。如果去掉这个注解，再尝试访问url，就会返回404。

也可以不使用注解，直接在web.xml中配置：

```xml
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>com.itranswarp.learnjava.servlet.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>

```

跑起来之后，去webapps目录下观察一下，发现tomcat是将war包解压出来运行的。

web.xml路径：webapps\xxx\WEB-INF\web.xml

class文件路径：webapps\xxx\WEB-INF\classes

回忆了一下，之前的项目组打包的时候，并没有将所有class打成一个war或者jar，而是直接把class放到一个zip包中，在环境中将zip包解压。按tomcat需要的目录组织好，然后将bin目录软链接到机器上实际安装的tomcat目录。

从而实现各个进程共用同一个tomcat的bin，但是conf目录下的配置却各不相同。



**conf/web.xml与webapps\xxx\WEB-INF\web.xml的关系**

<https://stackoverflow.com/questions/5937226/what-is-conf-web-xml-used-for-in-tomcat-as-oppsed-to-the-one-in-web-inf>

简单来说，由于webapps目录下面可以放多个项目。所以conf/web.xml代表的是全局配置，webapps\xxx\WEB-INF\web.xml代表的是war包特有的配置。



[Tomcat 配置文件详解](https://www.cnblogs.com/54chensongxia/p/13255055.html)

## idea集成tomcat

每次更改文件都重复编译war包，停止tomcat，复制war包，启动tomcat的过程，效率实在是太低了。

还好idea提供了集成tomcat的功能：

[IntelliJ IDEA – Run / debug web application on Tomcat](https://mkyong.com/intellij/intellij-idea-run-debug-web-application-on-tomcat/)

其中Deployment页面的context path决定了解压到webapps目录下的文件夹名称，也就决定了url的前缀。

不知道为啥，修改java文件之后，tomcat没办法自动重启。只有mvn compile重新打出war包之后，tomcat才会自动重启。所以只能每次手动点一下重启。



# spring mvc

spring mvc是基于Servlet的一个框架。为什么需要spring mvc呢，无非是让事情做起来更简单，更爽。

## 一个demo

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>spring-mvc-get-started</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.4</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.12.1</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>demo</finalName>
    </build>
</project>

```



src\main\webapp\WEB-INF\web.xml

```xml
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>

```



src\main\webapp\WEB-INF\springmvc-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- Provide support for component scanning -->
    <context:component-scan base-package="com.example"/>

    <!--Provide support for conversion, formatting and validation -->
    <mvc:annotation-driven/>

</beans>

```

src\main\java\com\example\DemoController.java

```java
package com.example;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(value = "/v1")
public class DemoController {
    @GetMapping(value = "/")
    public String Hello(@RequestParam String name, @RequestHeader String age) {
        return String.format("hello, %s, you are %s now!", name, age);
    }
}

```

貌似springmvc打出的war包不能命名为ROOT，否则请求一直报404



## spring mvc相比于直接使用servlet的优点

### uri组合更容易，从请求中获取参数更容易

使用Servlet时，从请求中获取参数，以及body对象：

```java
package com.itranswarp.learnjava.servlet;

import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(urlPatterns = "/")
public class HelloServlet extends HttpServlet {

    private static final long serialVersionUID = -2325044380107359765L;

    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    private static class Person {
        public String name;

        public String age;

        public String sex;
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter pw = resp.getWriter();

        int len = req.getContentLength();
        ServletInputStream iii = req.getInputStream();
        byte[] buffer = new byte[len];
        int i = iii.read(buffer, 0, len);

        String requestId = req.getParameter("requestId");
        Person person = OBJECT_MAPPER.readValue(buffer, Person.class);
        pw.write(String.format("requestId: %s\nname:%s\nage:%s\nsex:%s\n",
            requestId,
            person.name,
            person.age,
            person.sex));
        pw.flush();
    }
}

```

使用spring mvc：

```java
package com.example;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(value = "/v1")
public class DemoController {

    private static class Person {
        public String name;

        public String age;

        public String sex;
    }

    @PostMapping(value = "/")
    public String Hello(@RequestParam String requestId, @RequestBody Person person) {
        return String.format("requestId: %s\nname:%s\nage:%s\nsex:%s\n",
            requestId,
            person.name,
            person.age,
            person.sex);
    }
}

```

使用spring mvc，可以在class上定义base path，在method上再定义sub path，组合起来更为方便。

获取参数变得更简单是非常明显的。不用直接感知底层的HttpServletRequest/HttpServletResponse，连反序列化body也不需要手动调用json库进行操作了。

### 校验RequestParam

<https://www.baeldung.com/spring-validate-requestparam-pathvariable>

实测：

1. 添加MethodValidationPostProcessor bean

   ```java
   @Configuration
   public class ClientWebConfigJava implements WebMvcConfigurer {
       @Bean
       public MethodValidationPostProcessor methodValidationPostProcessor() {
           return new MethodValidationPostProcessor();
     	}
   }
   ```

2. 在Controller的class上添加注解：@Validated 

3. 在参数后面添加参数校验的注解

   ```java
       @PostMapping(value = "/")
       public String hello(
           @RequestParam @Size(min = 5, max = 100) String requestId,
           @Valid @RequestBody Person person) {
           return String.format("requestId: %s\nname:%s\nage:%s\nsex:%s\n",
               requestId,
               person.name,
               person.age,
               person.sex);
       }
   ```

   参数校验不通过返回的是500，默认错误信息：

   ```
   类型 异常报告
   
   消息 Request processing failed; nested exception is javax.validation.ConstraintViolationException: hello.arg0: 个数必须在5和100之间
   
   描述 服务器遇到一个意外的情况，阻止它完成请求。
   
   例外情况
   ....
   ```

   

### 校验RequestBody

1. 定义bean时加上注解

   ```java
       private static class Person {
           @Size(min = 5, max = 10, message = "name must be size 5-10")
           public String name;
   
           public String age;
   
           public String sex;
       }
   ```

2. 参数前添加@Valid注解即可。

不需要MethodValidationPostProcessor bean，也不需要在class上添加@Validated 注解。

参数校验不通过返回的是400，默认错误信息：

```
HTTP状态 400 - 错误的请求
类型 状态报告

描述 由于被认为是客户端对错误（例如：畸形的请求语法、无效的请求信息帧或者虚拟的请求路由），服务器无法或不会处理当前请求。
```

这部分还不知道如何定制。



### 直接返回model

```java
    @PostMapping(value = "/", consumes = MediaType.APPLICATION_JSON_VALUE)
    public Person Hello(@RequestParam String requestId, @RequestBody Person person) {
        return person;
    }
}
```

springmvc框架会自动对model进行json序列化。

### 接口文档与代码同源

结合swagger，做到接口文档与代码同源，可以认为最重要的优点之一。用了springmvc，还在方法中直接返回string/Object/Map/JSONObject的，简直就是在乱搞。

添加依赖：

```
<!--生成swagger文档-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

SwaggerConfig.java

```java
package com.example;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

@Configuration
@EnableOpenApi
@ComponentScan(basePackages = {"com.example"})
public class SwaggerConfig implements WebMvcConfigurer {

    @Bean
    public Docket customDocket() {
        return new Docket(DocumentationType.SWAGGER_2).
            select().
            apis(RequestHandlerSelectors.basePackage("com.example")).
            paths(PathSelectors.any()).
            build().apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
            .title("test")
            .contact(new Contact("小明", "http://www.cnblogs.com/getupmorning/", "zhaoming0018@126.com"))
            .version("1.1.0")
            .build();
    }

}

```

实测跑起来之后，可以从http://localhost:8082/springmvc/v3/api-docs访问json格式的swagger文件（springmvc是war包名）

v2也可以：http://localhost:8082/springmvc/v2/api-docs

但是：http://localhost:8082/springmvc/swagger-ui/index.html却没办法访问ui界面，不知道为啥。先不管了。



> https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api

#### 编译时生成swagger文档

必须把代码跑起来才能获取swagger文档吗？文档应该随着代码编译自动就生成出来。

找到一个[swagger-maven-plugin](https://github.com/kongchen/swagger-maven-plugin) 配置有点麻烦....后面再试



## tomcat servlet与spring 怎么关联起来的

通过web.xml中配置servlet为spring mvc提供的：org.springframework.web.servlet.DispatcherServlet（org.springframework:spring-webmvc）

tomcat启动时将加载这个servlet，在这个servlet中将启动Ioc容器，管理所有的bean。所有请求都由DispatcherServlet分发到不同的controller。

controller中使用的GetMaping PostMaping都是spring支持的。

> [廖雪峰-使用Spring MVC](https://www.liaoxuefeng.com/wiki/1252599548343744/1282383921807393)
>
> [controller类与servlet的关系](https://zhuanlan.zhihu.com/p/66934627)



# spring boot MVC

## spring-boot

spring mvc，需要安装tomcat，需要配置好tomcat的配置文件。整个jvm进程的main函数是在tomcat之中。

spring-boot，目的就是简化配置，直接内置tomcat，0配置，main函数就是业务代码中，一个jar包，所有东西就直接跑起来了。

## demo

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>springboot-mvc-get-started</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```



Application.java

```java
package com.example.servingwebcontent;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```



HelloController.java

```java
package com.example.servingwebcontent.controller;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    private static class Person {
        public String name;

        public String age;

        public String sex;
    }

    @PostMapping(value = "/hello")
    public String hello(String requestId, @RequestBody Person person) {
        return String.format("requestId: %s\nname:%s\nage:%s\nsex:%s\n",
            requestId,
            person.name,
            person.age,
            person.sex);
    }
}

```



就这三个文件，直接运行Application.java，一个web程序就跑起来了。

spring-boot-starter-web引入了tomcat-embed-core，所以无需再安装tomcat，也无需web.xml等配置，spring-boot都直接给配置好了。

spring-boot-maven-plugin默认将工程打包为一个jar，在环境上，java -jar 运行，一个web服务就跑起来了。





