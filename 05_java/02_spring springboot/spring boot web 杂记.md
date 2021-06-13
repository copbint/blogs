[toc]

# application.yml  vs application.properties

同时存在时，两个配置文件都有效，但是如果同一项配置，两个配置文件中都有，application.properties中的配置项优先级更高。

# 添加一个Filter

```java
package com.example.servingwebcontent.filter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
@Order(2)
public class RequestResponseLoggingFilter implements Filter {
    private static final Logger LOG = LoggerFactory.getLogger(RequestResponseLoggingFilter.class);

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;
        LOG.info("Logging Request  {} : {}", req.getMethod(), req.getRequestURI());
        chain.doFilter(request, response);
        LOG.info("Logging Response :{}", res.getContentType());
    }
}
```

 <https://www.baeldung.com/spring-boot-add-filter>

说实话，还搞不清这几个filter的关系。



tomcat filter

spring mvc filter

spring webflux filter

 



