title: 定制Gin的Validator错误消息及响应结构
date: 2021-11-14 19:02:00
toc: true
category:
- Golang
- Gin
tags:
- Golang
- golang
- Go
- go
- Gin
- gin
- Validator
- validator
- 功能
- 校验
- 定制
- 实现
- 响应
- 结构
- 错误
- 消息
- 本地化
- 翻译
---

## 前言

在使用 Golang 的 Validator(校验) 功能时，会发现很多错误的提示信息都是英文的。例如

```json
{
    "code": -1,
    "msg": {
        "SingUpForm.Age": "Key: 'SingUpForm.Age' Error:Field validation for 'Age' failed on the 'gte' tag",
        "SingUpForm.Email": "Key: 'SingUpForm.Email' Error:Field validation for 'Email' failed on the 'required' tag",
        "SingUpForm.Name": "Key: 'SingUpForm.Name' Error:Field validation for 'Name' failed on the 'required' tag",
        "SingUpForm.Password": "Key: 'SingUpForm.Password' Error:Field validation for 'Password' failed on the 'required' tag",
        "SingUpForm.RePassword": "Key: 'SingUpForm.RePassword' Error:Field validation for 'RePassword' failed on the 'required' tag"
    }
}
```

这样会造成接口调用者的阅读困难。
翻阅官方文档后发现翻译的工作已经做过了，可以直接将错误信息修改为指定语言。

但需要注意的是，目前 `go-playground/validator` 只支持了 13 种语言，具体参阅官方列出的目录：[validator/translations at master · go-playground/validator](https://github.com/go-playground/validator/tree/master/translations)

![Snipaste20211114190832.png](https://b3logfile.com/file/2021/11/Snipaste_2021-11-14_19-08-32-0b319fb0.png)

## 错误消息本地化

直接上代码，逻辑很清晰。每次出现错误信息通过 `err.(validator.ValidationErrors)` 转换得到异常对象 `validator.ValidationErrors`，调用其 `func (ve ValidationErrors) Translate(ut ut.Translator) ValidationErrorsTranslations {}` 方法即可。

```golang
package main

import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/locales/en"
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"

	en_translations "github.com/go-playground/validator/v10/translations/en"
	zh_translations "github.com/go-playground/validator/v10/translations/zh"
)

// 翻译校验信息
func main() {
	if err := InitTranslate("zh"); err != nil {
		fmt.Println("初始化翻译器出错")
		return
	}

	r := gin.Default()

	r.POST("/", func(c *gin.Context) {
		var singUpForm SingUpForm
		if err := c.ShouldBind(&singUpForm); nil != err {
			// 出现错误
			errors, ok := err.(validator.ValidationErrors)
			if !ok {
				c.JSON(http.StatusOK, gin.H{
					"code": -1,
					"msg":  fmt.Errorf("error getting translation of error message, %s", err.Error()),
				})
				return
			}

			c.JSON(http.StatusBadRequest, gin.H{
				"code": -1,
				"msg":  errors.Translate(translator),
			})

			return
		}

		c.JSON(http.StatusOK, gin.H{
			"code": 0,
			"msg":  "登录成功",
		})
	})

	// 监听并在 0.0.0.0:8080 上启动服务
	r.Run(":8080")
}

type SingUpForm struct {
	Age        uint8  `json:"age" binding:"gte=1,lte=130"`
	Name       string `json:"name" binding:"required,min=3"`
	Email      string `json:"email" binding:"required,email"`
	Password   string `json:"password" binding:"required"`
	RePassword string `json:"rePassword" binding:"required,eqfield=Password"`
}

// use a single instance , it caches struct info
var (
	uni        *ut.UniversalTranslator
	validate   *validator.Validate
	translator ut.Translator
)

func InitTranslate(locale string) (err error) {
	// 修改 gin 框架中的 validator 引擎属性，实现翻译效果
	engine, ok := binding.Validator.Engine().(*validator.Validate)
	if !ok {
		fmt.Println("获取 gin 框架的 validator 出错")
	}

	zhT := zh.New()
	enT := en.New()

	uni := ut.New(enT, zhT, enT)

	translator, ok = uni.GetTranslator(locale)
	if !ok {
		return fmt.Errorf("uni.GetTranslator(%s)", locale)
	}

	switch locale {
	case "en":
		en_translations.RegisterDefaultTranslations(engine, translator)
	case "zh":
		zh_translations.RegisterDefaultTranslations(engine, translator)
	}

	return
}
```

修改后的效果为：

```json
{
    "code": -1,
    "msg": {
        "SingUpForm.Age": "Age必须大于或等于1",
        "SingUpForm.Email": "Email为必填字段",
        "SingUpForm.Name": "Name为必填字段",
        "SingUpForm.Password": "Password为必填字段",
        "SingUpForm.RePassword": "RePassword为必填字段"
    }
}
```

## 响应内容结构属性名称首字母小写

原来的响应内容：

```json
{
    "code": -1,
    "msg": {
        "SingUpForm.Age": "Age必须大于或等于1",
        "SingUpForm.Email": "Email为必填字段",
        "SingUpForm.Name": "Name为必填字段",
        "SingUpForm.Password": "Password为必填字段",
        "SingUpForm.RePassword": "RePassword为必填字段"
    }
}
```

修改后的内容为：

```json
{
    "code": -1,
    "msg": {
        "SingUpForm.age": "age必须大于或等于1",
        "SingUpForm.email": "email为必填字段",
        "SingUpForm.name": "name为必填字段",
        "SingUpForm.password": "password为必填字段",
        "SingUpForm.rePassword": "rePassword为必填字段"
    }
}
```

完整代码为：

```golang
package main

import (
	"fmt"
	"net/http"
	"reflect"
	"strings"

	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/locales/en"
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"

	en_translations "github.com/go-playground/validator/v10/translations/en"
	zh_translations "github.com/go-playground/validator/v10/translations/zh"
)

// 翻译校验信息
func main() {
	if err := InitTranslate("zh"); err != nil {
		fmt.Println("初始化翻译器出错")
		return
	}

	r := gin.Default()

	r.POST("/", func(c *gin.Context) {
		var singUpForm SingUpForm
		if err := c.ShouldBind(&singUpForm); nil != err {
			// 出现错误
			errors, ok := err.(validator.ValidationErrors)
			if !ok {
				c.JSON(http.StatusOK, gin.H{
					"code": -1,
					"msg":  fmt.Errorf("error getting translation of error message, %s", err.Error()),
				})
				return
			}

			c.JSON(http.StatusBadRequest, gin.H{
				"code": -1,
				"msg":  errors.Translate(translator),
			})

			return
		}

		c.JSON(http.StatusOK, gin.H{
			"code": 0,
			"msg":  "登录成功",
		})
	})

	// 监听并在 0.0.0.0:8080 上启动服务
	r.Run(":8080")
}

type SingUpForm struct {
	Age        uint8  `json:"age" binding:"gte=1,lte=130"`
	Name       string `json:"name" binding:"required,min=3"`
	Email      string `json:"email" binding:"required,email"`
	Password   string `json:"password" binding:"required"`
	RePassword string `json:"rePassword" binding:"required,eqfield=Password"`
}

// use a single instance , it caches struct info
var (
	uni        *ut.UniversalTranslator
	validate   *validator.Validate
	translator ut.Translator
)

func InitTranslate(locale string) (err error) {
	// 修改 gin 框架中的 validator 引擎属性，实现翻译效果
	engine, ok := binding.Validator.Engine().(*validator.Validate)
	if !ok {
		fmt.Println("获取 gin 框架的 validator 出错")
	}

	// 实现响应内容 tag 首字母小写
	engine.RegisterTagNameFunc(func(field reflect.StructField) string {
		// 如 RePassword string `json:"rePassword" binding:"required,eqfield=Password"`
		// 那么 jsonTag 就是 rePassword
		jsonTag := field.Tag.Get("json")
		// 分割，例如 strings.SplitN("a,b,c", ",", 2) 那么就是 [a b,c]，strings.SplitN("a,b,c,d", ",", 2) 那么就是 [a b c,d]
		splitN := strings.SplitN(jsonTag, ",", 2)
		name := splitN[0]
		if name == "-" {
			return ""
		}
		return name
	})

	zhT := zh.New()
	enT := en.New()

	uni := ut.New(enT, zhT, enT)

	translator, ok = uni.GetTranslator(locale)
	if !ok {
		return fmt.Errorf("uni.GetTranslator(%s)", locale)
	}

	switch locale {
	case "en":
		en_translations.RegisterDefaultTranslations(engine, translator)
	case "zh":
		zh_translations.RegisterDefaultTranslations(engine, translator)
	}

	return
}
```

## 移除响应内容中的结构体

原来的响应内容：

```json
{
    "code": -1,
    "msg": {
        "SingUpForm.Age": "Age必须大于或等于1",
        "SingUpForm.Email": "Email为必填字段",
        "SingUpForm.Name": "Name为必填字段",
        "SingUpForm.Password": "Password为必填字段",
        "SingUpForm.RePassword": "RePassword为必填字段"
    }
}
```

修改后的内容为：

```json
{
    "code": -1,
    "msg": {
        "age": "age必须大于或等于1",
        "email": "email为必填字段",
        "name": "name为必填字段",
        "password": "password为必填字段",
        "rePassword": "rePassword为必填字段"
    }
}
```

完整代码为：

```golang
package main

import (
	"fmt"
	"net/http"
	"reflect"
	"strings"

	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/locales/en"
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"

	en_translations "github.com/go-playground/validator/v10/translations/en"
	zh_translations "github.com/go-playground/validator/v10/translations/zh"
)

// 翻译校验信息
func main() {
	if err := InitTranslate("zh"); err != nil {
		fmt.Println("初始化翻译器出错")
		return
	}

	r := gin.Default()

	r.POST("/", func(c *gin.Context) {
		var singUpForm SingUpForm
		if err := c.ShouldBind(&singUpForm); nil != err {
			// 出现错误
			errors, ok := err.(validator.ValidationErrors)
			if !ok {
				c.JSON(http.StatusOK, gin.H{
					"code": -1,
					"msg":  fmt.Errorf("error getting translation of error message, %s", err.Error()),
				})
				return
			}

			c.JSON(http.StatusBadRequest, gin.H{
				"code": -1,
				"msg":  removeTopStruct(errors.Translate(translator)),
			})

			return
		}

		c.JSON(http.StatusOK, gin.H{
			"code": 0,
			"msg":  "登录成功",
		})
	})

	// 监听并在 0.0.0.0:8080 上启动服务
	r.Run(":8080")
}

type SingUpForm struct {
	Age        uint8  `json:"age" binding:"gte=1,lte=130"`
	Name       string `json:"name" binding:"required,min=3"`
	Email      string `json:"email" binding:"required,email"`
	Password   string `json:"password" binding:"required"`
	RePassword string `json:"rePassword" binding:"required,eqfield=Password"`
}

// use a single instance , it caches struct info
var (
	uni        *ut.UniversalTranslator
	validate   *validator.Validate
	translator ut.Translator
)

func InitTranslate(locale string) (err error) {
	// 修改 gin 框架中的 validator 引擎属性，实现翻译效果
	engine, ok := binding.Validator.Engine().(*validator.Validate)
	if !ok {
		fmt.Println("获取 gin 框架的 validator 出错")
	}

	// 实现响应内容 tag 首字母小写
	engine.RegisterTagNameFunc(func(field reflect.StructField) string {
		// 如 RePassword string `json:"rePassword" binding:"required,eqfield=Password"`
		// 那么 jsonTag 就是 rePassword
		jsonTag := field.Tag.Get("json")
		// 分割，例如 strings.SplitN("a,b,c", ",", 2) 那么就是 [a b,c]，strings.SplitN("a,b,c,d", ",", 2) 那么就是 [a b c,d]
		splitN := strings.SplitN(jsonTag, ",", 2)
		name := splitN[0]
		if name == "-" {
			return ""
		}
		return name
	})

	zhT := zh.New()
	enT := en.New()

	uni := ut.New(enT, zhT, enT)

	translator, ok = uni.GetTranslator(locale)
	if !ok {
		return fmt.Errorf("uni.GetTranslator(%s)", locale)
	}

	switch locale {
	case "en":
		en_translations.RegisterDefaultTranslations(engine, translator)
	case "zh":
		zh_translations.RegisterDefaultTranslations(engine, translator)
	}

	return
}

// 移除响应的 Struct 名称，例如 `SingUpForm.age` 会变成 `age`
func removeTopStruct(fields map[string]string) map[string]string {
	resp := make(map[string]string)
	for key, value := range fields {
		resp[key[strings.Index(key, ".")+1:]] = value
	}
	return resp
}
```

## 参考资料

- [自定义验证器 | Gin Web Framework](https://gin-gonic.com/zh-cn/docs/examples/custom-validators/)。
- [go-playground/validator: :100:Go Struct and Field validation, including Cross Field, Cross Struct, Map, Slice and Array diving](https://github.com/go-playground/validator)
- [validator/main.go at master · go-playground/validator](https://github.com/go-playground/validator/blob/master/_examples/translations/main.go)
- [validator/zh.go at master · go-playground/validator](https://github.com/go-playground/validator/blob/master/translations/zh/zh.go)
