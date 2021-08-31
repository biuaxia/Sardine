title: 【转载】当 Docker 遇到 Intellij IDEA，再次解放了生产力~
date: 2021-08-06 16:51:49
toc: true
category:
 - 转载
 - Java
tags: 
 - 转载
 - Docker
 - IDEA
 - 生产力
 - Java
---

本文转载自：[当 Docker 遇到 Intellij IDEA，再次解放了生产力~ | 芋道源码 —— 纯源码解析博客](https://www.iocoder.cn/Fight/Development-increases-productivity-tenfold-IDEA-remote-one-click-deployment-of-Spring-Boot-to-Docker/)

---

Idea是Java开发利器，springboot是Java生态中最流行的微服务框架，docker是时下最火的容器技术，那么它们结合在一起会产生什么化学反应呢？

<!-- more -->

## 开发前准备

### Docker的安装可以参考https://docs.docker.com/install/

### 配置docker远程连接端口

```bash
vi /usr/lib/systemd/system/docker.service
```

找到 **ExecStart** ，在最后面添加 **-H tcp://0.0.0.0:2375** ，如下图所示

![](https://b3logfile.com/file/2021/08/solo-fetchupload-8693237600646122383-e44cd4c1.jpeg)

### 重启docker

```bash
systemctl daemon-reload
systemctl start docker
```

### 开放端口

```bash
firewall-cmd --zone=public --add-port=2375/tcp --permanent
```

### Idea安装插件,重启

![](https://b3logfile.com/file/2021/08/solo-fetchupload-1898377888658368282-a5001f12.jpeg)

### 连接远程docker

### 编辑配置

![](https://b3logfile.com/file/2021/08/solo-fetchupload-8273894719427021964-c5dc35c6.jpeg)

### 填远程docker地址

![](https://b3logfile.com/file/2021/08/solo-fetchupload-208295432697970943-330ea53d.jpeg)

### 连接成功，会列出远程docker容器和镜像

![](https://b3logfile.com/file/2021/08/solo-fetchupload-3076349342138451980-0cb23231.jpeg)

## 新建项目

### 创建springboot项目

项目结构图

![](https://b3logfile.com/file/2021/08/solo-fetchupload-1457604761030527958-f4a93c7e.jpeg)

### 配置pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>docker-demo</groupId>
    <artifactId>com.demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
        <relativePath />
    </parent>

    <properties>
         <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
         <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
         <docker.image.prefix>com.demo</docker.image.prefix>
         <java.version>1.8</java.version>
    </properties>
    <build>
        <plugins>
          <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
        <plugin>
           <groupId>com.spotify</groupId>
           <artifactId>docker-maven-plugin</artifactId>
           <version>1.0.0</version>
           <configuration>
              <dockerDirectory>src/main/docker</dockerDirectory>
              <resources>
                <resource>
                    <targetPath>/</targetPath>
                    <directory>${project.build.directory}</directory>
                    <include>${project.build.finalName}.jar</include>
                </resource>
              </resources>
           </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-antrun-plugin</artifactId>
            <executions>
                 <execution>
                     <phase>package</phase>
                    <configuration>
                        <tasks>
                            <copy todir="src/main/docker" file="target/${project.artifactId}-${project.version}.${project.packaging}"></copy>
                        </tasks>
                     </configuration>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    </execution>
            </executions>
        </plugin>

       </plugins>
    </build>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
  <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
</project>
```

### 在src/main目录下创建docker目录，并创建Dockerfile文件

```
FROM openjdk:8-jdk-alpine
ADD *.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

### 在resource目录下创建application.properties文件

```properties文件
logging.config=classpath:logback.xml
logging.path=/home/developer/app/logs/
server.port=8990
```

### 创建DockerApplication文件

```java
@SpringBootApplication
public class DockerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DockerApplication.class, args);
    }
}
```

### 创建DockerController文件

```java
@RestController
public class DockerController {
    static Log log = LogFactory.getLog(DockerController.class);

    @RequestMapping("/")
    public String index() {
        log.info("Hello Docker!");
        return "Hello Docker!";
    }
}
```

### 增加配置

![](https://b3logfile.com/file/2021/08/solo-fetchupload-6894516613402638944-89fa56e3.jpeg)

![](https://b3logfile.com/file/2021/08/solo-fetchupload-6182867014888705021-62f5e723.jpeg)

![](https://b3logfile.com/file/2021/08/solo-fetchupload-2111096551668017525-27cef861.jpeg)

命令解释  **Image tag :** 指定镜像名称和**tag** ，镜像名称为 **docker-demo** ，**tag** 为**1.1**  **Bind ports :** 绑定宿主机端口到容器内部端口。格式为[宿主机端口]:[容器内部端口]  **Bind mounts :** 将宿主机目录挂到到容器内部目录中。格式为[宿主机目录]:[容器内部目录]。这个springboot项目会将日志打印在容器 **/home/developer/app/logs/** 目录下，将宿主机目录挂载到容器内部目录后，那么日志就会持久化容器外部的宿主机目录中。

### Maven打包

![](https://b3logfile.com/file/2021/08/solo-fetchupload-4415082072267711542-8ae15486.jpeg)

### 运行

![](https://b3logfile.com/file/2021/08/solo-fetchupload-4426359272790922578-0381da9c.jpeg)

![](https://b3logfile.com/file/2021/08/solo-fetchupload-8417262683625205734-ac540089.jpeg)

先pull基础镜像，然后再打包镜像，并将镜像部署到远程docker运行

![](https://b3logfile.com/file/2021/08/solo-fetchupload-1158557468327292938-3ee5e599.jpeg)

这里我们可以看到镜像名称为docker-demo:1.1，docker容器为docker-server

### 运行成功

![](https://b3logfile.com/file/2021/08/solo-fetchupload-3621286957594217481-57ee40c2.jpeg)

### 浏览器访问

![](https://b3logfile.com/file/2021/08/solo-fetchupload-68605866890662950-ddb1900e.jpeg)

### 日志查看

![](https://b3logfile.com/file/2021/08/solo-fetchupload-2009835354866706005-0f0edd8a.jpeg)

自此通过idea 部署springboot项目到docker成功！难以想象，部署一个Javaweb项目竟然如此简单方便！
