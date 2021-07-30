title: 【转载】golang设置桌面壁纸
date: '2021-07-06 11:14:45'
updated: '2021-07-07 10:32:36'
category:
 - 转载
 - Go
tags: [转载, Go]
permalink: /articles/2021/07/06/1625541285156.html
---

本文转载自：[golang设置桌面壁纸 - Go语言中文网 - Golang中文社区](https://studygolang.com/topics/11815) & [Golang版更换windows 壁纸为 bing每日背景图 - Go语言中文网 - Golang中文社区](https://studygolang.com/articles/31251)

---

## 版本1

golang语言编写，设置windows桌面壁纸，图片来自于必应网站（cn.bing.com）


<!-- more -->


github源码地址：[https://github.com/tujiaw/gowallpaper](https://github.com/tujiaw/gowallpaper)
有兴趣的Star一下吧

编译好的程序：[https://pan.baidu.com/s/1l5OW9GeuUF0r5TFaBWkWZg](https://pan.baidu.com/s/1l5OW9GeuUF0r5TFaBWkWZg) (提取码：pcqh)

直接双击运行，显示如下：

```
设置微软必应的壁纸，用法如下：
day     - 每天更新壁纸
now     - 设置当天壁纸
prev    - 设置前一天壁纸
next    - 设置后一天壁纸
rand    - 间隔随机切换壁纸（如每分钟切换壁纸：rand 1m）
quit    - 退出
#
```

### golang调用windows API

```go
package winapi

import (
    "log"
    "syscall"
    "unsafe"
)

var ApiList = map[string][]string {
    "user32.dll": {
        "MessageBoxW",
        "SystemParametersInfoW",
    },
    "kernel32.dll": {

    },
}

var ProcCache map[string]*syscall.Proc

func init() {
    ProcCache = make(map[string]*syscall.Proc)
    for dllName, apiList := range ApiList {
        d, err := syscall.LoadDLL(dllName)
        if err != nil {
            panic(err)
        }
        for _, name := range apiList {
            api, err := d.FindProc(name)
            if err != nil {
                log.Println(err, name)
            }
            ProcCache[name] = api
        }
        _ = syscall.FreeLibrary(d.Handle)
    }
}

func WinCall(name string, a ...uintptr) {
    if api, ok := ProcCache[name]; ok {
        _, _, err := api.Call(a...)
        if err != nil {
            log.Println(err)
        }
    } else {
        log.Println("api not found, name:", name)
    }
}

func IntPtr(n int) uintptr {
    return uintptr(n)
}

func StrPtr(s string) uintptr {
    p, _ := syscall.UTF16PtrFromString(s)
    return uintptr(unsafe.Pointer(p))
}

func ShowMessage(title, text string) {
    WinCall("MessageBoxW", IntPtr(0), StrPtr(text), StrPtr(title), IntPtr(0))
}

func SetWallpaper(bmpPath string) {
    WinCall("SystemParametersInfoW", IntPtr(20), IntPtr(0), StrPtr(bmpPath), IntPtr(3))
}
```

---

## 版本2

直接从 bing 搜索主页获取图片链接, 修改链接中的大小, 获取图片, 调用windows的 dll 设置桌面背景图

```go
package main

import (
    "crypto/md5"
    "encoding/hex"
    "errors"
    "fmt"
    "github.com/antchfx/htmlquery"
    "io/ioutil"
    "net/http"
    "os"
    "path/filepath"
    "regexp"
    "strings"
    "syscall"
    "time"
    "unsafe"
)

const (
    UserAgent      = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36"
    BingHomeURL    = "https://cn.bing.com"
    CurrentPathDir = "cache/"
)

const (
    Size1k string = "1920,1080"
    Size2k string = "2560,1440"
    Size4k string = "3840,2160"
)

// ImageSize 图片大小
type ImageSize struct {
    w string
    h string
}

func init() {
    _ = os.Mkdir(CurrentPathDir, 0755)
}

// EncodeMD5 MD5编码
func EncodeMD5(value string) string {
    m := md5.New()
    m.Write([]byte(value))
    return hex.EncodeToString(m.Sum(nil))
}

// SetWindowsWallpaper 设置windows壁纸
func SetWindowsWallpaper(imagePath string) error {
    dll := syscall.NewLazyDLL("user32.dll")
    proc := dll.NewProc("SystemParametersInfoW")
    _t, _ := syscall.UTF16PtrFromString(imagePath)
    ret, _, _ := proc.Call(20, 1, uintptr(unsafe.Pointer(_t)), 0x1|0x2)
    if ret != 1 {
        return errors.New("系统调用失败")
    }
    return nil
}

// GetBingBackgroundImageURL 获取bing主页的背景图片链接
func GetBingBackgroundImageURL() (string, error) {
    client := http.Client{}

    request, err := http.NewRequest("GET", BingHomeURL, nil)
    if err != nil {
        return "", err
    }
    request.Header.Set("user-agent", UserAgent)

    response, err := client.Do(request)
    if err != nil {
        return "", err
    }

    htmlDoc, err := htmlquery.Parse(response.Body)
    if err != nil {
        return "", err
    }

    item := htmlquery.FindOne(htmlDoc, "//div[@id=\"bgImgProgLoad\"]")
    result := htmlquery.SelectAttr(item, "data-ultra-definition-src")
    return BingHomeURL + result, nil
}

// DownloadImage 下载图片,保存并返回保存的文件名的绝对路径
func DownloadImage(imageURL string, size *ImageSize) (string, error) {

    wRegexp := regexp.MustCompile("w=\\d+")
    hRegexp := regexp.MustCompile("h=\\d+")
    imageURL = wRegexp.ReplaceAllString(imageURL, "w="+size.w)
    imageURL = hRegexp.ReplaceAllString(imageURL, "h="+size.h)

    client := http.Client{}

    request, err := http.NewRequest("GET", imageURL, nil)
    if err != nil {
        return "", err
    }

    response, err := client.Do(request)
    if err != nil {
        return "", err
    }
    body, err := ioutil.ReadAll(response.Body)
    if err != nil {
        return "", err
    }

    day := time.Now().Format("2006-01-02")

    fileName := EncodeMD5(imageURL)
    path := CurrentPathDir + fmt.Sprintf("[%sx%s][%s]%s", size.w, size.h, day, fileName) + ".jpg"

    err = ioutil.WriteFile(path, body, 0755)
    if err != nil {
        return "", err
    }
    absPath, err := filepath.Abs(path)
    if err != nil {
        return "", err
    }

    return absPath, nil
}

func main() {
    fmt.Println("获取必应背景图中...")
    imageURL, err := GetBingBackgroundImageURL()
    if err != nil {
        fmt.Println("获取背景图片链接失败: " + err.Error())
        return
    }
    fmt.Println("获取成功: " + imageURL)

    fmt.Println("下载图片...")
    imagePath, err := DownloadImage(imageURL, &ImageSize{
        w: strings.Split(Size4k, ",")[0],
        h: strings.Split(Size4k, ",")[1],
    })
    if err != nil {
        fmt.Println("下载图片失败: " + err.Error())
        return
    }
    fmt.Println("设置桌面...")
    err = SetWindowsWallpaper(imagePath)
    if err != nil {
        fmt.Println("设置桌面背景失败: " + err.Error())
        return
    }
}
```

---

## 我的版本

```go
package main

import (
	"bytes"
	"crypto/md5"
	"encoding/hex"
	"errors"
	"fmt"
	"io/ioutil"
	"net/http"
	"os"
	"path/filepath"
	"regexp"
	"strings"
	"syscall"
	"time"
	"unsafe"

	"github.com/antchfx/htmlquery"
)

func main() {
	// fmt.Println("获取必应背景图中...")
	// imageURL, err := GetBingBackgroundImageURL()
	// if err != nil {
	// 	fmt.Println("获取背景图片链接失败: " + err.Error())
	// 	return
	// }
	// fmt.Println("获取成功: " + imageURL)

	fmt.Println("下载图片...")
	imagePath, err := DownloadImage(computerMobilePhoneAdaptiveRandomPicture, &ImageSize{
		w: strings.Split(Size4k, ",")[0],
		h: strings.Split(Size4k, ",")[1],
	})
	if err != nil {
		fmt.Println("下载图片失败: " + err.Error())
		return
	}
	fmt.Println("设置桌面...")
	err = SetWindowsWallpaper(imagePath)
	if err != nil {
		fmt.Println("设置桌面背景失败: " + err.Error())
		return
	}
}

const (
	UserAgent      = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36"
	BingHomeURL    = "https://cn.bing.com"
	CurrentPathDir = "cache/"
	RandBingImgURL = "https://img.tjit.net/bing/rand/"
	// 电脑手机自适应随机图片
	computerMobilePhoneAdaptiveRandomPicture = "https://img.tjit.net/bing/rand/"
)

const (
	Size1k string = "1920,1080"
	Size2k string = "2560,1440"
	Size4k string = "3840,2160"
)

// ImageSize 图片大小
type ImageSize struct {
	w string
	h string
}

func init() {
	_ = os.Mkdir(CurrentPathDir, 0755)
}

// EncodeMD5 MD5编码
func EncodeMD5(value string) string {
	m := md5.New()
	m.Write([]byte(value))
	return hex.EncodeToString(m.Sum(nil))
}

// SetWindowsWallpaper 设置windows壁纸
func SetWindowsWallpaper(imagePath string) error {
	dll := syscall.NewLazyDLL("user32.dll")
	proc := dll.NewProc("SystemParametersInfoW")
	_t, _ := syscall.UTF16PtrFromString(imagePath)
	ret, _, _ := proc.Call(20, 1, uintptr(unsafe.Pointer(_t)), 0x1|0x2)
	if ret != 1 {
		return errors.New("系统调用失败")
	}
	return nil
}

// GetBingBackgroundImageURL 获取bing主页的背景图片链接
func GetBingBackgroundImageURL() (string, error) {
	client := http.Client{}

	request, err := http.NewRequest("GET", BingHomeURL, nil)
	if err != nil {
		return "", err
	}
	request.Header.Set("user-agent", UserAgent)

	response, err := client.Do(request)
	if err != nil {
		return "", err
	}

	bodyRes, err := ioutil.ReadAll(response.Body)
	body := ioutil.NopCloser(bytes.NewReader(bodyRes))
	body1 := ioutil.NopCloser(bytes.NewReader(bodyRes))

	htmlDoc, err := htmlquery.Parse(body)
	if err != nil {
		return "", err
	}
	all, err := ioutil.ReadAll(body1)
	s := string(all)
	fmt.Println(s)
	item := htmlquery.FindOne(htmlDoc, "//div[@id=\"bgImgProgLoad\"]")
	result := htmlquery.SelectAttr(item, "data-ultra-definition-src")
	return BingHomeURL + result, nil
}

// DownloadImage 下载图片,保存并返回保存的文件名的绝对路径
func DownloadImage(imageURL string, size *ImageSize) (string, error) {

	wRegexp := regexp.MustCompile("w=\\d+")
	hRegexp := regexp.MustCompile("h=\\d+")
	imageURL = wRegexp.ReplaceAllString(imageURL, "w="+size.w)
	imageURL = hRegexp.ReplaceAllString(imageURL, "h="+size.h)

	client := http.Client{}

	request, err := http.NewRequest("GET", imageURL, nil)
	if err != nil {
		return "", err
	}

	response, err := client.Do(request)
	if err != nil {
		return "", err
	}
	body, err := ioutil.ReadAll(response.Body)
	if err != nil {
		return "", err
	}

	day := time.Now().Format("2006-01-02")

	fileName := EncodeMD5(imageURL)
	path := CurrentPathDir + fmt.Sprintf("[%sx%s][%s]%s", size.w, size.h, day, fileName) + ".jpg"

	err = ioutil.WriteFile(path, body, 0755)
	if err != nil {
		return "", err
	}
	absPath, err := filepath.Abs(path)
	if err != nil {
		return "", err
	}

	return absPath, nil
}
```
