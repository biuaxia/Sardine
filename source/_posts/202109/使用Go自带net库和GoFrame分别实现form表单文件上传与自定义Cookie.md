title: 使用Go自带net库和GoFrame分别实现form表单文件上传与自定义Cookie
date: 2021-09-17 13:26:00
toc: true
category:
- Golang
tags:
- Golang
- net
- http
- 库
- GoFrame
- 实现
- 表单
- 文件
- 上传
- 自定义
- Cookie
---

本文实现使用Go语言调用链滴图床。

下图展示的内容为两种方式分别上传同一张图片到链滴图床。

![image.png](https://b3logfile.com/file/2021/09/image-ed55eb98.png)

代码如下：

```golang
package main

import (
	"bytes"
	"context"
	"crypto/md5"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"io"
	"io/ioutil"
	"log"
	"mime/multipart"
	"net/http"
	"os"
	"path/filepath"
	"strings"

	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/os/glog"
)

const (
	// loginUrl 登录地址
	loginUrl = "https://ld246.com/login"

	// uploadImgUrl 图片上传地址
	uploadImgUrl = "https://ld246.com/upload/editor"
	// uploadImgUrl = "http://localhost:9999"

	// uploadFilePath 上传文件所在路径
	uploadFilePath = "C:/Users/biuaxia/Downloads/Snipaste_2021-09-07_15-03-20.png"
)

type loginReqData struct {
	NameOrEmail  string `json:"nameOrEmail"`
	UserPassword string `json:"userPassword"`
	Captcha      string `json:"captcha"`
}

type loginRespData struct {
	Msg       string `json:"msg"`
	Code      int    `json:"code"`
	Goto      string `json:"goto"`
	TokenName string `json:"tokenName"`
	Token     string `json:"token"`
}

// main [链滴](http://ld246.com)图床上传接口
func main() {
	data := getLoginRespData()

	uploadImg(data)
	uploadPic(data)
}

// uploadImg 使用 goFrame 实现文件上传
func uploadImg(data loginRespData) {
	m := make(map[string]string)
	m[data.TokenName] = data.Token

	r, e := g.Client().
		Cookie(m).
		Post(uploadImgUrl, "file[]=@file:"+uploadFilePath)
	if e != nil {
		glog.Error(e)
	} else {
		fmt.Println("uploadImg-resp:", string(r.ReadAll()))
		err := r.Close()
		if err != nil {
			panic(err)
		}
	}
}

// uploadPic 使用原生实现文件上传
// key:file 里面放一个文件
// multipart/form-data 传一个文件
func uploadPic(data loginRespData) {
	m := make(map[string]string)
	m[data.TokenName] = data.Token

	var (
		buffer = bytes.NewBuffer(nil)
		writer = multipart.NewWriter(buffer)
	)

	formFile, _ := writer.CreateFormFile("file[]", filepath.Base(uploadFilePath))
	file, _ := os.Open(uploadFilePath)
	_, _ = io.Copy(formFile, file)
	defer func(file *os.File) {
		err := file.Close()
		if err != nil {
			panic(err)
		}
	}(file)
	defer func(writer *multipart.Writer) {
		err := writer.Close()
		if err != nil {
			panic(err)
		}
	}(writer)

	req, _ := http.NewRequest("POST", uploadImgUrl, buffer)
	req.Header.Set("Content-Type", writer.FormDataContentType())
	req = req.WithContext(context.Background())
	req.Header.Set("Cookie", data.TokenName+"="+data.Token)
	req.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/93.0.4577.63 Safari/537.36 Edg/93.0.961.47")

	resp, _ := http.DefaultClient.Do(req)
	readAll, _ := ioutil.ReadAll(resp.Body)
	fmt.Println("uploadPic-resp:", string(readAll))
}

// getLoginRespData 获取登录信息，主要是响应得到的 Token，用来拼接图床请求的 Cookie 信息
func getLoginRespData() loginRespData {
	ld := loginReqData{
		NameOrEmail:  "你的账号",
		UserPassword: getPassword("你的账号"),
		Captcha:      "",
	}

	// 准备请求内容
	reqBody, _ := json.Marshal(ld)
	log.Println("请求内容:", string(reqBody))

	resp, _ := http.Post(loginUrl, "application/json", strings.NewReader(string(reqBody)))
	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			panic(err)
		}
	}(resp.Body)

	body, _ := ioutil.ReadAll(resp.Body)
	log.Println("响应内容:", string(body))

	var lrd loginRespData
	_ = json.Unmarshal(body, &lrd)
	return lrd
}

// getPassword 获取 Md5 加密后的内容
func getPassword(password string) string {
	h := md5.New()
	h.Write([]byte(password))
	return hex.EncodeToString(h.Sum(nil))
}
```
