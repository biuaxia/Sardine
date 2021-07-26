title: 致远解决“您没有在符合的IP范围内登录”
date: '2021-07-08 09:44:27'
updated: '2021-07-08 10:12:31'
category:
 - 致远
tags: [致远, 解决]
permalink: /articles/2021/07/08/1625708667297.html
---
## 已验证

- A8_V80：√

---

数据库执行语句：

```sql
select 
	* 
from 
	ipcontrol; 
delete from 
	ipcontrol;
```

重启服务即可。

## 已失效

数据库执行语句：

```sql
UPDATE ctp_config set CONFIG_VALUE = 'disable' where CONFIG_ITEM = 'ip_control_enable'; 
```

重启服务即可。
