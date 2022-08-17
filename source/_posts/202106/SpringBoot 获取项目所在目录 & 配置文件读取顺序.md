title: SpringBoot 获取项目所在目录 & 配置文件读取顺序
date: 2021-06-21 16:22:12
toc: true
category: 
 - Java
tags: 
 - SpringBoot
 - Java
 - 获取
 - 项目
 - 目录
 - 配置
 - 文件
 - 读取
 - 顺序
---

项目目录与当前类所在目录：

```java
log.info("项目目录: {}", System.getProperty("user.dir"));
log.info("项目目录: {}", environment.getProperty("user.dir"));
log.info("当前类所在目录: {}", ClassUtils.
        getDefaultClassLoader().
        getResource(CharSequenceUtil.EMPTY).
        getPath());
log.info("启动类所在目录: {}", ResourceUtils.
        getURL("classpath:").
        getPath());
```

项目目录如果是以 jar 启动则是 jar 所在目录，如果是用代码直接启动就在项目根目录下。

直接启动：

![image.png](https://b3logfile.com/file/2021/06/image-281541f9.png)

命令行通过 `java -jar x.jar` 启动：

![image.png](https://b3logfile.com/file/2021/06/image-3f738f26.png)

结构为：

![image.png](https://b3logfile.com/file/2021/06/image-15d41cca.png)

---

以jar包发布springboot项目时，默认会先使用jar包跟目录下的application.properties来作为项目配置文件。

如果在不同的目录中存在多个配置文件，它的读取顺序是：

1. config/application.properties（项目根目录中config目录下）
2. config/application.yml
3. application.properties（项目根目录下）
4. application.yml
5. resources/config/application.properties（项目resources目录中config目录下）
6. resources/config/application.yml
7. resources/application.properties（项目的resources目录下）
8. resources/application.yml

> 注：
>
> 1. 如果同一个目录下，有application.yml也有application.properties，默认先读取application.properties。
> 2. 如果同一个配置属性，在多个配置文件都配置了，默认使用第1个读取到的，后面读取的不覆盖前面读取到的。
> 3. 创建SpringBoot项目时，一般的配置文件放置在“项目的resources目录下”