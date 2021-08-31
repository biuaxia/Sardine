title: Landray FAQ
date: 2021-06-18 23:31:35
toc: true
category: 
 - Landray
tags: 
 - Landray
 - FAQ
---

## 🙌 说明

查找答案时请善用搜索功能。


<!-- more -->


## 考核一、👦 新手入门

请严格按照下面步骤搭建环境，出现问题时第一时间排除环境方面的原因。

> 蓝凌EKPv14-v15，GKP1.0 仅支持 Oracle-JDK1.7.79 以上（仅限JDK7）。

> 同理，Tomcat也仅支持 **7**。

1. 下载 **Oracle-JDK**
2. 下载 **Tomcat**
3. 下载 **Eclipse**（为了项目修改内容的发版签名），无版本要求
4. 下载 **idea**，无版本要求

如果不知道如何下载或者百度也帮不到你，可以直接从百度云盘下载 JDK+Eclipse+Tomcat 的**整合包**。

> `biuaxia`更新于 __2021年6月2日16点02分__

> 链接：`https://pan.baidu.com/s/1lMJ2oeUDFpNVrs3eRkcWfw`

> 提取码：6ix1

环境搭建好后，请将下面的 GKP 项目导入到 IDE 中（可以是 Eclipse，也可以是 IDEA）。

> `biuaxia`更新于 __2021年6月2日16点12分__

> 链接：`https://pan.baidu.com/s/1pLy3OJIB60yGO4OgIY8Ljg`

> 提取码：4ft3

以web项目启动，启动后默认的访问账号和密码是：admin/1

## 考核二、

### 🆘 IDEA Build 报错 OutOfMemoryError ？

![](https://b3logfile.com/file/2021/06/solo-fetchupload-1572353435686932751-aca5d6cb.png )

如图，进入设置（Ctrl + Alt + S）`File | Settings | Build, Execution, Deployment | Compiler` 修改下图中的数字为你电脑内存的一半，建议大于等于4G。

![](https://b3logfile.com/file/2021/06/solo-fetchupload-776351722478398677-26375a08.png )

### IDEA 启动报错 OutOfMemoryError: **PermGen space？**

将Tomcat按照下图，在 `VM options` 中添加：

```bash
-Xms1024m -Xmx8192m -XX:PermSize=1024m -XX:MaxPermSize=8192m
```

其中`1024`代表最小，`8192`代表最大，内存将在此区间波动。请修改为符合你电脑硬件配置的值再次启动即可。

![](https://b3logfile.com/file/2021/06/solo-fetchupload-521307315504617139-90a24903.png )

## 🔍 问答

### 修改主题样式?

文件路径为`扩展主题包\iceblue.zip\style\widget.css`。

### 门户页面如何去掉头像？

进入`项目部署目录/sys/portal/page`下，修改下面的jsp即可。如果有多个则表示配置了多个页面内容。

例如`/sys/portal/page/175f8359d71b9b95d30603a48ec95cc1`这样的文件夹，`175f8359d71b9b95d30603a48ec95cc1`是`门户引擎 > 门户维护 > 页面配置` 对应页面的 fdId。

他的进入链接为`/sys/portal/sys_portal_page/sysPortalPage.do?method=view&fdId=175f8359d71b9b95d30603a48ec95cc1`。

修改头像则打开对应的 JSP 页面，注释掉下面代码的 7-16 行即可。

```java
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ include file="/sys/ui/jsp/common.jsp"%>
<%@ include file="/sys/portal/sys_portal_page/page.jsp"%>
<%@ taglib uri="/WEB-INF/KmssConfig/sys/portal/portal.tld" prefix="portal"%>
<template:include ref="template.t" pagewidth="980px">  
 <template:replace name="aside"> 
  <ui:nonepanel layout="sys.ui.nonepanel.default" scroll="false" id="p_70be49095f7701f2b273">
   <portal:portlet title="个人头像">
    <ui:dataview format="sys.ui.html">
     <ui:source ref="sys.person.head.tab.source"></ui:source>
     <ui:render ref="sys.ui.html.default"></ui:render>
     <ui:var name="showNoDataTip" value="true"></ui:var>
     <ui:var name="showErrorTip" value="true"></ui:var>
    </ui:dataview>
   </portal:portlet>
  </ui:nonepanel> 
  <ui:nonepanel layout="sys.ui.nonepanel.default" scroll="false" id="p_aaba7467c267250e45dc">
```

变成这样即可。

```java
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ include file="/sys/ui/jsp/common.jsp"%>
<%@ include file="/sys/portal/sys_portal_page/page.jsp"%>
<%@ taglib uri="/WEB-INF/KmssConfig/sys/portal/portal.tld" prefix="portal"%>
<template:include ref="template.t" pagewidth="980px">  
 <template:replace name="aside"> 
<%--  <ui:nonepanel layout="sys.ui.nonepanel.default" scroll="false" id="p_70be49095f7701f2b273">
   <portal:portlet title="个人头像">
    <ui:dataview format="sys.ui.html">
     <ui:source ref="sys.person.head.tab.source"></ui:source>
     <ui:render ref="sys.ui.html.default"></ui:render>
     <ui:var name="showNoDataTip" value="true"></ui:var>
     <ui:var name="showErrorTip" value="true"></ui:var>
    </ui:dataview>
   </portal:portlet>
  </ui:nonepanel> --%>
  <ui:nonepanel layout="sys.ui.nonepanel.default" scroll="false" id="p_aaba7467c267250e45dc">
```
