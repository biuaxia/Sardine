title: SpringBoot中使用JUnit进行代码测试
date: 2021-08-24 15:30:00
toc: true
category:
 - Java
tags:
 - Java
 - SpringBoot
 - 单元
 - 测试
---

Junit 是 Java 知名的测试框架，通过集成 Junit + SpringBoot Test 可以在测试程序中使用 SpringBoot 的依赖注入等 SpringBoot 特性。

## 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```

## 测试 Service

位置：`src/main/java/com/example/service/impl/HelloServiceImpl.java`

```java
package com.example.service.impl;

@Service
public class HelloServiceImpl implements HelloService {

    @Override
    public String helloTo(String name) {
        // 故意写一个 Bug
        if ("C++".equals(name))
            return "Don't hello to C++";
        return StringUtils.hasText(name) ? "HELLO " + name : "HELLO WORLD";
    }
}
```

想要测试这一代码，Java 通用的方式自己 `new` 出所需要的资源，再进行测试使用，但这样的话就无法使用 SpringBoot 的各种特性了，如果需要的依赖特别多，或者是牵扯到各种配置读取的话，简直就是噩梦：

```java
private HelloService helloService;

@Before
public void setUp() {
    helloService = new HelloService();
}

@Test
public void helloTo() {
    assertEquals("HELLO WORLD", helloService.helloTo(""));
    assertEquals("HELLO WORLD", helloService.helloTo("WORLD"));
    assertEquals("HELLO JAVA", helloService.helloTo("JAVA"));
    assertEquals("HELLO C++", helloService.helloTo("C++")); // 出错
}
```

### 方法一：注解启动 SpringBoot

位置：`src/test/java/com/example/service/impl/HelloServiceImplTest.java`

重点：

* 使用 Junit SpringBoot Test 框架：`@RunWith(SpringRunner.class)`
* 指定启动入口：`@SpringBootTest(classes = [SpringBoot 程序入口类])`

在测试结束之后，SpringBoot 上下文会自动关闭。

```java
package com.example.service.impl;

import com.example.SpringBootTestApplication;
import com.example.service.HelloService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringBootTestApplication.class)
public class HelloServiceImplTest {

    @Autowired
    private HelloService helloService;

    @Test
    public void helloTo() {
        assertEquals("HELLO WORLD", helloService.helloTo(""));
        assertEquals("HELLO WORLD", helloService.helloTo("WORLD"));
        assertEquals("HELLO JAVA", helloService.helloTo("JAVA"));
        assertEquals("HELLO C++", helloService.helloTo("C++")); // 出错
    }
}
```

### 方法二：显式启动 SpringBoot

通过使用 `SpringApplication::run` 方法来启动 SpringBoot 程序，也可以做到使用 SpringBoot 所有特性来进行测试，但是稍微麻烦了些，而且必须在测试结束之后手动对 `SpringBootContext` 进行关闭。

```java
package com.example.service.impl;

import com.example.SpringBootTestApplication;
import com.example.service.HelloService;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.springframework.boot.SpringApplication;
import org.springframework.context.ConfigurableApplicationContext;

import static org.junit.Assert.assertEquals;

public class HelloServiceImplTest2 {

    private ConfigurableApplicationContext context;
    private HelloService helloService;

    @Before
    public void setUp() {
        context = SpringApplication.run(SpringBootTestApplication.class);
        // 显式获取 Bean，所需依赖 SpringBoot 会自动注入
        helloService = context.getBean(HelloService.class);
    }

    @After
    public void tearDown() {
        // 关闭 SpringBoot 程序
        context.close();
    }

    @Test
    public void helloTo() {
        assertEquals("HELLO WORLD", helloService.helloTo(""));
        assertEquals("HELLO WORLD", helloService.helloTo("WORLD"));
        assertEquals("HELLO JAVA", helloService.helloTo("JAVA"));
        assertEquals("HELLO C++", helloService.helloTo("C++")); // 出错
    }
}
```
