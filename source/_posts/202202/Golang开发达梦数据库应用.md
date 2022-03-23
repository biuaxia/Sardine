title: Golang开发达梦数据库应用
date: 2022-02-21 14:23:13
toc: true
index_img: https://b3logfile.com/file/2022/02/23a5526744174569ada00a90ca3167f3.png
category:
- Golang
tags:
- Golang
- 开发
- 达梦
- 数据库
- 应用
---

## Go 数据库接口

[Go 语言标准库 database/sql](https://golang.google.cn/pkg/database/sql/) 提供了一系列数据库操作的标准接口，DM 数据库基于 GO 1.13 版本通过实现 database/sql 包的接口，向开发人员提供 DM 数据库操作的 Go 语言接口。

## 开发环境搭建

| 软件      | 版本                             |
| --------- | -------------------------------- |
| DM 数据库 | DM 8.0 及以上版本                |
| GO        | Go 1.13 Windows-amd 64           |
| LiteIDE   | LiteIDE x20.1，Go 语言开发编辑器 |
| GIT       | Git-2.29.2-64-bit                |

### 安装 DM 数据库

请参考 [DM 数据库快速上手指南](https://eco.dameng.com/docs/zh-cn/start/index)。

数据库安装过程中，请勾选创建 `BOOKSHOP`，`DMHR` 示例库，作为数据库模拟环境，如下图所示：

![DMHR 示例库](https://b3logfile.com/file/2022/02/23a5526744174569ada00a90ca3167f3.png)

### 安装 Go 环境

可从[官网下载](https://studygolang.com/dl) Go 1.13 Windows-amd 64 安装包，也有解压版下载，如下图所示：

![安装包图片](https://b3logfile.com/file/2022/02/62478c9e68e643449448c05aa5e6eae7.png)

### 安装 Go 语言编辑器 LiteIDE

LiteIDE 是一个轻量级 Go 语言开发编辑器，下载 [LiteIDE 安装包](https://download.dameng.com/eco/docs/liteidex20.1.windows.7z "LiteIDE安装包")。

### 下载并安装 Git

请[下载并安装 Git](https://git-scm.com/download/win)，通过 Git 获取驱动程序。

Git 安装步骤简单，持续点击【下一步】即可。安装完成后，在桌面单击鼠标右键，选择【Git Bash Here】按钮，出现下图即表明安装成功。

![Git 安装成功](https://b3logfile.com/file/2022/02/bab21e6d0eb04811b763c0111db0dd15.png)

### DM 驱动包的配置

1. 安装 DM 驱动包

  将提供的 DM 驱动包放在 GOPATH 的 src 目录下。驱动包位于 `dmdbms/drivers/go/dm-go-driver.zip` 解压到 GPPATH 安装路径的 src 下，如图所示：

  ![安装dm驱动包](https://b3logfile.com/file/2022/02/0f4faef2eeec4526b6d8c643eb42f144.png)

2. 安装依赖包

所需 Go 依赖包有两个，`text` 和 `snappy`，从 Git 上把依赖包 clone 到本地，桌面右键【Git Bash Here】打开 Git 命令行窗口，依次下载 text 和 snappy 依赖。

```
git clone https://github.com/golang/text.git  D:/Go/src/golang.org/x/text
git clone https://github.com/golang/snappy  D:/Go/src/github.com/golang/snappy
```

> **注意**示例安装路径为 `D:/Go` 注意换成自己的安装路径。

  ![clone 依赖包](https://b3logfile.com/file/2022/02/3726ed60fdd248a7874d9c3f605cc343.png)

  ![clone 依赖包](https://b3logfile.com/file/2022/02/bd1ae918727d4fe8967068662b6e7b29.png)

## 绑定变量及大字段操作示例

打开 LiteIDE 编辑工具，新建一个目录，并在目录下新建一 TestDemo.go 文件。

```golang
/*该例程实现插入数据，修改数据，删除数据，数据查询等基本操作。*/
package main

// 引入相关包
import (
	"database/sql"
	"dm"
	"fmt"
	"io/ioutil"
	"time"
)

var db *sql.DB
var err error

func main() {
	driverName := "dm"
	dataSourceName := "dm://SYSDBA:SYSDBA@localhost:5236"
	if db, err = connect(driverName, dataSourceName); err != nil {
		fmt.Println(err)
		return
	}
	if err = insertTable(); err != nil {
		fmt.Println(err)
		return
	}
	if err = updateTable(); err != nil {
		fmt.Println(err)
		return
	}
	if err = queryTable(); err != nil {
		fmt.Println(err)
		return
	}
	if err = deleteTable(); err != nil {
		fmt.Println(err)
		return
	}
	if err = disconnect(); err != nil {
		fmt.Println(err)
		return
	}
}

/* 创建数据库连接 */
func connect(driverName string, dataSourceName string) (*sql.DB, error) {
	var db *sql.DB
	var err error
	if db, err = sql.Open(driverName, dataSourceName); err != nil {
		return nil, err
	}
	if err = db.Ping(); err != nil {
		return nil, err
	}
	fmt.Printf("connect to \"%s\" succeed.\n", dataSourceName)
	return db, nil
}

/* 往产品信息表插入数据 */
func insertTable() error {
	var inFileName = "D:\\三国演义.jpg"
	var sql = `INSERT INTO production.product(name,author,publisher,publishtime,
product_subcategoryid,productno,satetystocklevel,originalprice,nowprice,discount,
description,photo,type,papertotal,wordtotal,sellstarttime,sellendtime)
VALUES(:1,:2,:3,:4,:5,:6,:7,:8,:9,:10,:11,:12,:13,:14,:15,:16,:17);`
	data, err := ioutil.ReadFile(inFileName)
	if err != nil {
		return err
	}
	t1, _ := time.Parse("2006-Jan-02", "2005-Apr-01")
	t2, _ := time.Parse("2006-Jan-02", "2006-Mar-20")
	t3, _ := time.Parse("2006-Jan-02", "1900-Jan-01")
	_, err = db.Exec(sql, "三国演义", "罗贯中", "中华书局", t1, 4, "9787101046126", 10, 19.0000, 15.2000,
		8.0,
		"《三国演义》是中国第一部长篇章回体小说，中国小说由短篇发展至长篇的原因与说书有关。",
		data, "25", 943, 93000, t2, t3)
	if err != nil {
		return err
	}
	fmt.Println("insertTable succeed")
	return nil
}

/* 修改产品信息表数据 */
func updateTable() error {
	var sql = "UPDATE production.product SET name = :name WHERE productid = 11;"
	if _, err := db.Exec(sql, "三国演义（上）"); err != nil {
		return err
	}
	fmt.Println("updateTable succeed")
	return nil
}

/* 查询产品信息表 */
func queryTable() error {
	var productid int
	var name string
	var author string
	var description dm.DmClob
	var photo dm.DmBlob
	var sql = "SELECT productid,name,author,description,photo FROM production.product WHERE productid=11"
	rows, err := db.Query(sql)
	if err != nil {
		return err
	}
	defer rows.Close()
	fmt.Println("queryTable results:")
	for rows.Next() {
		if err = rows.Scan(&productid, &name, &author, &description, &photo); err != nil {
			return err
		}
		blobLen, _ := photo.GetLength()
		fmt.Printf("%v %v %v %v %v\n", productid, name, author, description, blobLen)
	}
	return nil
}

/* 删除产品信息表数据 */
func deleteTable() error {
	var sql = "DELETE FROM production.product WHERE productid = 12;"
	if _, err := db.Exec(sql); err != nil {
		return err
	}
	fmt.Println("deleteTable succeed")
	return nil
}

/* 关闭数据库连接 */
func disconnect() error {
	if err := db.Close(); err != nil {
		fmt.Printf("db close failed: %s.\n", err)
		return err
	}
	fmt.Println("disconnect succeed")
	return nil
}
```

> **注意**提前在 `D:\\三国演义.jpg` 准备好要插入的大字段图片。

点击【build and run】按钮运行代码，运行截图如下:

![运行后截图](https://b3logfile.com/file/2022/02/6785292ebb9a44f88fefd6d85daac079.png)

运行后数据库截图如下图所示：

![运行后数据库截图](https://b3logfile.com/file/2022/02/7fd6402ed2c94846a07cf526407b9055.png)

## 参考资料

- [达梦 Go 驱动源码 | 达梦数据库](https://eco.dameng.com/download/)，翻到最下面找到 **资源下载**。
