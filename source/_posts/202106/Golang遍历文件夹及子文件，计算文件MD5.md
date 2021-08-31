title: Golang遍历文件夹及子文件，计算文件MD5
date: 2021-06-22 11:04:03
toc: true
category: 
 - Golang
tags: 
 - Go
 - Golang
 - 遍历
 - 文件夹
 - 子文件
 - 计算
 - 文件
 - MD5
 - TODO
---

> 2021年6月29日11点55分更新代码
>
> 运行后将会在当前目录生成文件："当前目录名.txt" 的文件


<!-- more -->


```go
package main

import (
	"crypto/md5"
	"encoding/hex"
	"fmt"
	"io"
	"io/fs"
	"io/ioutil"
	"os"
	"path/filepath"
)

// main 交叉编译命令：
// go tool dist list
/*
x86_x64的Linux: SET CGO_ENABLED=0 SET GOOS=linux SET GOARCH=amd64 go build hello.go
国产化arm: SET CGO_ENABLED=0 SET GOOS=linux SET GOARCH=arm64 go build hello.go
// go build -o hello.go
*/
func main() {
	startPath, _ := os.Getwd()
	fmt.Println("startPath:", startPath)

	paths, err := readDir(startPath)
	if nil != err {
		panic(err)
	}

	fmt.Printf("当前目录共计 \"%d\" 个文件\n", len(paths))

	// 获取文件 md5
	outputContent := ""
	for _, path := range paths {
		hash := md5.New()
		open, err := os.Open(path)
		if nil != err {
			panic(err)
		}

		_, _ = io.Copy(hash, open)

		sum := hash.Sum(nil)
		md5Str := hex.EncodeToString(sum)
		singleLine := fmt.Sprintf("%s -> %s\n", md5Str, path)
		outputContent += singleLine
	}

	// 准备写入文件
	fileName := fmt.Sprintf("%s.txt", filepath.Base(startPath))
	err = ioutil.WriteFile(fileName, []byte(outputContent), 0644)
	if err != nil {
		fmt.Printf("%s\n", err.Error())
	} else {
		fmt.Printf("请查看当前目录下的 \"%s\" 文件\n", fileName)
	}

}

func readDir(pathname string) (paths []string, err error) {
	finalPath := pathname

	dir, err := ioutil.ReadDir(finalPath)
	if nil != err {
		return paths, err
	}

	for _, v := range dir {
		filename := v.Name()

		fullPath := fmt.Sprintf("%s%s%s", pathname, string(os.PathSeparator), filename)
		if v.IsDir() {
			err := filepath.WalkDir(fullPath, func(path string, d fs.DirEntry, err error) error {
				if nil != d && !d.IsDir() {
					paths = append(paths, path)
				}
				return err
			})
			if nil != err {
				fmt.Printf("%s\n", err.Error())
				return paths, err
			}
		} else {
			paths = append(paths, fullPath)
		}
	}

	return paths, nil
}
```

---

代码如下：

```go
package main

import (
	"crypto/md5"
	"encoding/hex"
	"fmt"
	"io"
	"io/fs"
	"io/ioutil"
	"os"
	"path/filepath"
)

func main() {
	paths, err := readDir("C:\\Users\\biuaxia\\go\\src\\awesomeProject")
	if nil != err {
		panic(err)
	}

	fmt.Printf("%d\n", len(paths))

	for _, path := range paths {
		hash := md5.New()
		open, err := os.Open(path)
		if nil != err {
			panic(err)
		}

		_, _ = io.Copy(hash, open)

		sum := hash.Sum(nil)
		md5Str := hex.EncodeToString(sum)
		fmt.Printf("%s -> %s\n", md5Str, path)
	}
}

func readDir(pathname string) (paths []string, err error) {
	finalPath := pathname

	dir, err := ioutil.ReadDir(finalPath)
	if nil != err {
		return paths, err
	}

	for _, v := range dir {
		filename := v.Name()
		fullPath := fmt.Sprintf("%s\\%s", pathname, filename)
		if v.IsDir() {
			err := filepath.WalkDir(fullPath, func(path string, d fs.DirEntry, err error) error {
				if !d.IsDir() {
					paths = append(paths, path)
				}
				return err
			})
			if nil != err {
				panic(err)
				return paths, err
			}
		} else {
			paths = append(paths, fullPath)
		}
	}

	return paths, nil
}
```

目录结构为：

![image.png](https://b3logfile.com/file/2021/06/image-5f32caa7.png)

运行结果为：

![image.png](https://b3logfile.com/file/2021/06/image-8d8bf74a.png)

附上 Windows 编译后的程序：

[http://ddns.biuaxia.cn:8005/s/plUJ](http://ddns.biuaxia.cn:8005/s/plUJ)，密码：f12pf9

## TODO

修复非 windows/amd64 操作系统无法使用的问题。

## 编译

```bash
SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build hello.go
```

## 查看当前 Go 支持编译的平台

```go
go tool dist list
```
