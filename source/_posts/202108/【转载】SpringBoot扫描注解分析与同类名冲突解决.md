title: 【转载】SpringBoot扫描注解分析与同类名冲突解决
date: 2021-08-24 15:02:22
toc: true
category:
 - Java
tags:
 - 转载
 - Java
 - SpringBoot
 - 注解
 - 分析
 - 场景
 - 冲突
 - 解决
 - 扫描
 - 项目
 - 模式
 - pom
 - 启动
 - 功能
---

本文转载自：[SpringBoot包扫描之多模块多包名扫描和同类名扫描冲突解决｜8月更文挑战](https://juejin.cn/post/6999428902275940382?utm_source=biuaxia.cn)

---

我们在开发`springboot`项目时候，创建好SpringBoot项目就可以通过启动类直间启动，运行一个web项目，非常方便简单，不像我们之前通过`Spring+SpringMvc`要运行启动一个web项目还需要要配置各种包扫描和`tomcat`才能启动

我将应用分成了parent+common+component+app这种模式

1. parent是一个单纯的pom文件，存放项目的一些公共依赖
2. common则是一个没有启动类的SpringBoot项目，存放项目的核心公共代码
3. component各种组件功能服务模块，用的时候直接引用插拔方式实现
4. app则是一个实际的应用项目，包含一个SpringBoot启动类，提供各种实际的功能。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-3926770392394474232-785cdd23.webp)


<!-- more -->


![](https://b3logfile.com/file/2021/08/solo-fetchupload-3983595921017405791-d5a9e375.webp)

其中`kmall-admin`和`kmall-api`则是一个实际的应用项目，包含一个SpringBoot启动类

> 但是我在启动项目时候发现 有些模块并未成功注入，配置类也没有生效。即SpringBoot并没有扫描到这些文件

# 场景分析

## SpringBoot 默认扫描机制

由于SpringBoot默认包扫描机制是：从`启动类`所在包开始，扫描当前包及其子包下的所有文件。

由于刚开始我的启动类包名为：`cn.soboys.kmall.admin.WebApplication`,而其他项目文件包名均为`cn.soboys.kmall.*.XxxClass`，故其他模块被引用时候下面文件无法被扫描注入。

## SpringBoot启动类的扫描注解

SpringBoot 启动类上，配置扫描包路径有三种方式，最近看到一个应用上三种注解都用上了，代码如下：

```java
@SpringBootApplication(scanBasePackages ={"a","b"})
@ComponentScan(basePackages = {"a","b","c"})
@MapperScan({"XXX"})
public class XXApplication extends SpringBootServletInitializer 
}
```

> 那么，疑问来了：SpringBoot 中，这三种注解生效优先级如何、第一种和第二种有没有区别呢？

# SpringBootApplication 注解

这是 SpringBoot 的注解，本质是三个 Spring 注解的和 看源码可知道

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.beans.factory.support.BeanNameGenerator;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.context.TypeExcludeFilter;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "nameGenerator"
    )
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```

1. @SpringBootConfiguration
2. @EnableAutoConfiguration
3. @ComponentScan

它默认扫描启动类所在包及其所有子包，但是不包括第三方的 jar 包的其他目录，通过scanBasePackages 属性可以重新设置扫描包路径。

# ComponentScan注解

这个是 `Spring` 框架的注解，它用来指定组件扫描路径，如果用这个注解，它的值必须包含整个工程中全部需要扫描的路径。因为它会`覆盖 SpringBootApplication` 的默认扫描路径，导致其失效。

失效表现有两种：

1. 如果 ComponentScan 只包括一个值且就是默认启动类目录，SpringBootApplication 生效， ComponentScan 注解失效，报错：
2. 如果 ComponentScan 指定多个具体子目录，此时 SpringBootApplication 会失效，Spring 只会扫描 ComponentScan 指定目录下的注解。如果恰好有目录外的 Controller 类，很遗憾，这些控制器将无法访问。

# MapperScan 注解

1. 这里又涉及到`@Mapper`注解

直接在`Mapper类`上面添加注解`@Mapper`，这种方式要求每一个`mapper类`都需要添加此注解，比较麻烦

2. 通过使用`@MapperScan`可以指定要扫描的Mapper类的包的路径

这个是 `MyBatis` 的注解，会将指定目录下所有 Mapper 类封装成 MyBatis 的 BaseMapper类，生成对应的`XxxMapper` 代理接口实现类然后注入 Spring 容器中，不需要额外的注解，就可以完成注入。

```java
//使用@MapperScan注解多个包

@SpringBootApplication  
@MapperScan({"com.kfit.demo","com.kfit.user"})  
public class App {  
    public static void main(String[] args) {  
       SpringApplication.run(App.class, args);  
    }  
}
```

如果如果mapper类没有在Spring Boot主程序可以扫描的包或者子包下面，可以使用如下方式进行配置：

```java
@SpringBootApplication  
@MapperScan({"com.kfit.*.mapper","org.kfit.*.mapper"})  
public class App {  
    public static void main(String[] args) {  
       SpringApplication.run(App.class, args);  
    }  
}
```

# 场景解决

所以分析了上面所有注解，我们有两种解决办法：

1. 通过`@SpringBootApplication`指定`scanBasePackages`属性可以重新设置扫描包路径
   
   ```java
   @SpringBootApplication(scanBasePackages = {"cn.soboys.kmall"},nameGenerator = UniqueNameGenerator.class)
   @MapperScan(value = {"cn.soboys.kmall.mapper","cn.soboys.kmall.sys.mapper",
           "cn.soboys.kmall.security.mapper","cn.soboys.kmall.monitor.mapper"},nameGenerator = UniqueNameGenerator.class)
   public class WebApplication {
       private static ApplicationContext applicationContext;
   
       public static void main(String[] args) {
           applicationContext =
                   SpringApplication.run(WebApplication.class, args);
           //displayAllBeans();
       }
   
   
       /**
        * 打印所以装载的bean
        */
       public static void displayAllBeans() {
           String[] allBeanNames = applicationContext.getBeanDefinitionNames();
           for (String beanName : allBeanNames) {
               System.out.println(beanName);
           }
       }
   }
   ```
2. 通过`@ComponentScan`指定`basePackages`属性指定组件扫描路径
   
   ```java
   @ComponentScan(basePackages =  {"cn.soboys.kmall"},nameGenerator = UniqueNameGenerator.class)
   @MapperScan(value = {"cn.soboys.kmall.mapper","cn.soboys.kmall.sys.mapper",
           "cn.soboys.kmall.security.mapper","cn.soboys.kmall.monitor.mapper"},nameGenerator = UniqueNameGenerator.class)
   public class WebApplication {
       private static ApplicationContext applicationContext;
   
       public static void main(String[] args) {
           applicationContext =
                   SpringApplication.run(WebApplication.class, args);
           //displayAllBeans();
       }
   
   
       /**
        * 打印所以装载的bean
        */
       public static void displayAllBeans() {
           String[] allBeanNames = applicationContext.getBeanDefinitionNames();
           for (String beanName : allBeanNames) {
               System.out.println(beanName);
           }
       }
   }
   ```

> 当然我们看到其中扫描中还指定了属性`nameGenerator`是为了解决在多模块，多包名，下`相同类名，扫描注入冲突问题`

spring提供两种`beanName`生成策略，基于注解的sprong-boot默认使用的是AnnotationBeanNameGenerator，它生成beanName的策略就是，取当前类名（不是全限定类名）作为beanName。由此，如果出现不同包结构下同样的类名称，肯定会出现冲突

> 解决方法：我们可以自己写一个类实现 `org.springframework.beans.factory.support.BeanNameGeneraot接口` 重新定义beanName生成策略，继承AnnotationBeanNameGenerator，重写generateBeanName

同样 解决mybatis不同包下面同名mapper bean名重复的问题

```java
public class UniqueNameGenerator extends AnnotationBeanNameGenerator {
    @Override
    public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {

        //全限定类名

        String beanName = definition.getBeanClassName();

        return beanName;

    }

}
```

```java
public class UniqueNameGenerator extends AnnotationBeanNameGenerator {

    @Override
    public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {

        //如果有设置了value，则用value，如果没有则是用全类名
        if (definition instanceof AnnotatedBeanDefinition) {
            String beanName = determineBeanNameFromAnnotation((AnnotatedBeanDefinition) definition);
            if (StringUtils.hasText(beanName)) {
                // Explicit bean name found.
                return beanName;
            }else{
                //全限定类名
                beanName = definition.getBeanClassName();
                return beanName;
            }
        }

        // 使用默认类名
        return buildDefaultBeanName(definition, registry);
    }
}
```

这种是限定全类名，也就是`包名+类名`

```java
package com;
 
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationBeanNameGenerator;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;
 
@Component("myNameGenerator")
public class MyNameGenerator extends AnnotationBeanNameGenerator {
    @Override
    protected String buildDefaultBeanName(BeanDefinition definition) {
        String beanClassName = definition.getBeanClassName();
        Assert.state(beanClassName != null, "No bean class name set");
        //分割类全路径
        String[] packages = beanClassName.split("\\.");
        StringBuilder beanName = new StringBuilder();
        //取类的包名的首字母小写再加上类名作为最后的bean名
        for (int i = 0; i < packages.length - 1; i++) {
            beanName.append(packages[i].toLowerCase().charAt(0));
        }
        beanName.append(packages[packages.length - 1]);
        return beanName.toString();
    }
}
```

这种是取类的`包名的首字母小写`再加上`类名`作为最后的bean名

2. 通过单独指定冲突类名的扫描名字来解决

在两个同名类上`@Service`注解或者`@controller`注解扫描的时候指定value值，

3. @Primary注解

这个注解就是为了解决当有多个bean满足注入条件时，有这个注解的实例被选中

## 参考文献

1. [扫描注解的用法及冲突原则](https://juejin.cn/post/6892493290671702030)
2. [同名类冲](https://blog.csdn.net/ZYC88888/article/details/84758835)
