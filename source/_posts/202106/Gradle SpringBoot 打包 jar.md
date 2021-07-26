title: Gradle SpringBoot 打包 jar
date: '2021-06-21 16:29:50'
updated: '2021-06-21 21:29:47'
category: 
 - Java
tags: [SpringBoot, gradle, 打包, jar]
permalink: /articles/2021/06/21/1624264190323.html
---
命令如下：

```bash
gradlew clean && gradlew bootJar
```

![image.png](https://b3logfile.com/file/2021/06/image-d4ace6e2.png)

执行后就在当前项目的 `build/libs` 下生成了 jar 文件。

![image.png](https://b3logfile.com/file/2021/06/image-30bd6d8e.png)
