title: fyne 支持中文字体永久解决方案
date: 2021-09-07 14:32:00
toc: true
category:
 - Golang
 - fyne
tags:
 - Golang
 - fyne
 - 支持
 - 中文
 - 字体
 - 永久
 - 解决
 - 方案
---

## 起因

当前版本的 fyne 对中文支持不是很友好，尝试一段代码调用 fyne 来展示中文，效果如图：

![image.png](https://b3logfile.com/file/2021/09/image-ebae4657.png)

代码为：

```go
package main

import (
	"fyne.io/fyne/v2"
	"fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/dialog"
	"fyne.io/fyne/v2/layout"
	"fyne.io/fyne/v2/widget"
)

func main() {
	a := app.New()
	w := a.NewWindow("font")
	w.Resize(fyne.NewSize(300, 200))
	w.SetContent(
		fyne.NewContainerWithLayout(
			layout.NewVBoxLayout(),
			layout.NewSpacer(),
			widget.NewLabel("撒旦撒旦"),
			widget.NewLabel("委屈额为"),
			widget.NewButton("请问犬瘟热第三方", func() {
				dialog.ShowInformation("非官方的", "从v风格化风格化", w)
			}),
			layout.NewSpacer(),
		),
	)
	w.ShowAndRun()
}
```

现在想要 fyne 支持中文需要详细阅读官方文档([Text | Develop using Fyne](https://developer.fyne.io/tour/canvas/text.html))才能够得到答案。

## fyne 的中文支持

文档中给出了两种方案：

1. 使用环境变量 `FYNE_FONT` 指定 `.ttf` 类型字体，参见：[Text | Develop using Fyne](https://developer.fyne.io/tour/canvas/text.html)
2. 捆绑字体文件（即自定义主题），参见：[Creating a Custom Theme | Develop using Fyne](https://developer.fyne.io/tutorial/custom-theme)

### 使用环境变量 `FYNE_FONT`

使用 fyne 内置的主题时，如果环境变量指定了字体文件，则可以将其用作字体。

#### Windows下指定环境变量

Windows 下通过设置环境变量 `FYNE_FONT` 为当前系统所支持的中文字体，例如 `华文隶书 常规` 字体，通过右键属性的方式得到字体名字：

![image.png](https://b3logfile.com/file/2021/09/image-7698b55f.png)

然后设置环境变量：

![image.png](https://b3logfile.com/file/2021/09/image-ed928b3f.png)

#### Linux 与其它Unix操作系统指定环境变量

```shell
FYNE_FONT=STLITI.TTF go run main.go
```

---

运行代码就能实现中文内容的显示了。

![image.png](https://b3logfile.com/file/2021/09/image-f6160939.png)

代码如下：

```go
package main

import (
	"fyne.io/fyne/v2"
	"fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/dialog"
	"fyne.io/fyne/v2/layout"
	"fyne.io/fyne/v2/widget"
)

func main() {
	a := app.New()
	w := a.NewWindow("font")
	w.Resize(fyne.NewSize(300, 200))
	w.SetContent(
		fyne.NewContainerWithLayout(
			layout.NewVBoxLayout(),
			layout.NewSpacer(),
			widget.NewLabel("撒旦撒旦"),
			widget.NewLabel("委屈额为"),
			widget.NewButton("请问犬瘟热第三方", func() {
				dialog.ShowInformation("非官方的", "从v风格化风格化", w)
			}),
			layout.NewSpacer(),
		),
	)
	w.ShowAndRun()
}
```

#### 环境变量方式的缺点

虽然看起来还不错，但是这种方式在分发编译后的程序时会出现许多问题。

## 捆绑字体文件（自定义主题）

fyne提供了默认的主题，由于主题的可设置项包括了字体，于是可以通过自定义主题来使用任何的字体。

自定义主题可以通过实现[主题接口](https://pkg.go.dev/fyne.io/fyne#Theme)来得到。

### 字体与 go 代码的捆绑

例如将字体 `常规体` 和 `粗体` 进行绑定。

命令为：

```shell
fyne bundle STLITI.ttf > bundle.go
fyne bundle -append STLITI-bold.ttf >> bundle.go
```

#### `fyne bundle` 命令

在 GUI 应用程序中，想要使用图像作为按钮的图标。 如果要分发应用程序就可以将资源文件（如图像文件）合并为单个二进制文件。

Fyne 为此提供了执行此操作的机制和有用的命令。

`fyne bundle` 通过使用命令，将资源文件（如图像）转换为 Fyne 可以处理的数据（具有字节原始数据的结构）。

具体可以参考：[https://github.com/fyne-io/fyne/tree/master/cmd/fyne](https://github.com/fyne-io/fyne/tree/master/cmd/fyne)

### 定义主题`Struct`

定义想用使用的主题 `struct` 名称，并且实现 `fyne.Theme` 接口。

其中 `Font` 请按照业务逻辑进行处理即可。

```go
type Theme interface {
	Color(ThemeColorName, ThemeVariant) color.Color
	Font(TextStyle) Resource
	Icon(ThemeIconName) Resource
	Size(ThemeSizeName) float32
}
```

代码如下：

```go
package main

import (
	"image/color"

	"fyne.io/fyne/v2"
	"fyne.io/fyne/v2/theme"
)

type Biu struct{}

func (b Biu) Font(s fyne.TextStyle) fyne.Resource {
	if s.Monospace {
		return theme.DefaultTheme().Font(s)
	}
	if s.Bold {
		if s.Italic {
			return theme.DefaultTheme().Font(s)
		}
		return resourceSTLITITTF
	}
	if s.Italic {
		return theme.DefaultTheme().Font(s)
	}
	return resourceSTLITITTF
}

func (b Biu) Color(name fyne.ThemeColorName, variant fyne.ThemeVariant) color.Color {
	return theme.DefaultTheme().Color(name, variant)
}

func (b Biu) Icon(name fyne.ThemeIconName) fyne.Resource {
	return theme.DefaultTheme().Icon(name)
}

func (b Biu) Size(name fyne.ThemeSizeName) float32 {
	return theme.DefaultTheme().Size(name)
}
```

### 使用自定义主题

通过 `fyne.App` 的 `Settings().SetTheme()` 方法实现，例如：

```go
package main

import (
	"fyne.io/fyne/v2"
	"fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/dialog"
	"fyne.io/fyne/v2/layout"
	"fyne.io/fyne/v2/widget"
)

func main() {
	a := app.New()
	w := a.NewWindow("font")
	a.Settings().SetTheme(&Biu{})
	w.Resize(fyne.NewSize(300, 200))
	w.SetContent(
		fyne.NewContainerWithLayout(
			layout.NewVBoxLayout(),
			layout.NewSpacer(),
			widget.NewLabel("撒旦撒旦"),
			widget.NewLabel("委屈额为"),
			widget.NewButton("请问犬瘟热第三方", func() {
				dialog.ShowInformation("非官方的", "从v风格化风格化", w)
			}),
			layout.NewSpacer(),
		),
	)
	w.ShowAndRun()
}
```

其中的 `a.Settings().SetTheme(&Biu{})` 就是具体的调用方式，运行结果为：

![image.png](https://b3logfile.com/file/2021/09/image-db046fd6.png)

## 完整代码下载

- [biu.zip](https://b3logfile.com/file/2021/09/biu-48ec9aad.zip)

## 参考资料

- [Fyne で日本語フォントを扱う | blog - lusingander](https://lusingander.netlify.app/posts/200614-fyne-font/)
- [Fyne でのアイコンや画像の取り扱い | blog - lusingander](https://lusingander.netlify.app/posts/200613-fyne-resourece/)
