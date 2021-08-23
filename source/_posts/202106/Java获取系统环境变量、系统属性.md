title: Java获取系统环境变量、系统属性
date: '2021-06-21 15:44:17'
toc: truue
permalink: /articles/2021/06/21/1624261456912.html
category: 
 - Java
tags: 
 - Java
 - env
 - 环境变量
---

## 系统环境变量

即配置在系统变量中的内容：

![image.png](https://b3logfile.com/file/2021/06/image-8c0990b5.png)


<!-- more -->

代码为：

```java
System.getEnv();
System.getEnv("OS");
System.getEnv("JAVA_HOME");
```

## 系统属性

JVM启动时的一些信息，例如：

![image.png](https://b3logfile.com/file/2021/06/image-a7b768a2.png)

代码如下：

```java
System.getProperties();
System.getProperty("os.name");
```

## 获取 yml/properties 配置文件信息

```java
package cn.biuaxia.code.core.interceptor;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.EnvironmentAware;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;

@Slf4j
@Configuration
public class AppStartupInterceptor implements  EnvironmentAware {

    /**
     * Set the {@code Environment} that this component runs in.
     *
     * @param environment
     */
    @Override
    public void setEnvironment(Environment environment) {
        log.info("cn.biuaxia.code.core.interceptor.AppStartupInterceptor.setEnvironment");
    }
}
```

其中的 `environment` 变量通过：

```java
environment.getProperty("spring.datasource.username")
```

就可以拿到：

![image.png](https://b3logfile.com/file/2021/06/image-eed6907f.png)

## 获取 Environment 的两种方式

非静态方法调用可以直接 @Autowired 或者 构造方法注入，例如：

```java
package cn.biuaxia.code.core.interceptor;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 全局拦截
 */
@Slf4j
@Configuration
public class GlobalInterceptor implements WebMvcConfigurer {

    private final Environment environment;

    public GlobalInterceptor(Environment environment) {
        this.environment = environment;
    }

    /**
     * Add Spring MVC lifecycle interceptors for pre- and post-processing of
     * controller method invocations and resource handler requests.
     * Interceptors can be registered to apply to all requests or be limited
     * to a subset of URL patterns.
     *
     * @param registry 拦截器注册表
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        String property = environment.getProperty("spring.datasource.url");
        log.info("cn/biuaxia/code/core/interceptor/GlobalInterceptor.java:property: [{}]", property);
        WebMvcConfigurer.super.addInterceptors(registry);
    }

}
```

---

对于静态方法可以 实现 `org.springframework.context.EnvironmentAware` 类的 `setEnvironment` 方法，例如：

```java
package cn.biuaxia.code;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.EnvironmentAware;
import org.springframework.core.env.Environment;

import java.util.HashMap;

/**
 * 项目启动引擎
 *
 * @author biuaxia
 */
@Slf4j
@SpringBootApplication
public class Engine implements EnvironmentAware {

    private static Environment environment;

    /**
     * Set the {@code Environment} that this component runs in.
     *
     * @param environment
     */
    @Override
    public void setEnvironment(Environment environment) {
        Engine.environment = environment;
    }

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Engine.class);
        app.setDefaultProperties(new HashMap<>());
        app.run(args);

        log.info(environment.getProperty("spring.datasource.url"));
        log.info(environment.getProperty("spring.datasource.username"));
        log.info(environment.getProperty("spring.datasource.password"));
    }
}
```
