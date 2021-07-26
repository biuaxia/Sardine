title: 【转载】更新数据库时报错 Data truncation Incorrect datetime value
date: '2021-07-13 10:44:46'
updated: '2021-07-13 10:44:46'
category:
 - 转载
tags: [MySQL, 错误, 解决]
permalink: /articles/2021/07/13/1626144286389.html
---
## 解决办法

对应数据表列的时间类型由`timestamp`改成`datetime` 就行了。

## 原理

数据库中时间类型的原因，导致这样的错误`datetime` 以 `YYYY-MM-DD HH:MM:SS` 格式检索和显示 `DATETIME` 值。支持的范围为 `1000-01-01 00:00:00` 到 `9999-12-31 23:59:59` 而 `TIMESTAMP` 值支持的范围 `1970-01-01 08:00:01` 到 `2038-01-19 11:14:07` 储存,对于 `TIMESTAMP` 来说如果不在这个范围就会报这个错。
