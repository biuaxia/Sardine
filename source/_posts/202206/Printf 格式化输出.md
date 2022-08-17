title: Printf 格式化输出
date: 2022-06-19 21:01:00
toc: false
index_img: http://api.btstu.cn/sjbz/?lx=m_dongman&cid=6
category:
- Go
tags:
- 读取
- 顺序
- 支持
- 格式
- 输出
- 文本
- 字符串
- 不同
- 打印
- 方法
- 指针
- 函数
---

Print、Println 、Printf 、Sprintf 、Fprintf都是fmt 包中的公共方法，在需要打印信息时需要用到这些函数，那么这些函数有什么区别呢？

```
Print: 输出到控制台(不接受任何格式化，它等价于对每一个操作数都应用 %v)
fmt.Print(str)
Println: 输出到控制台并换行
fmt.Println(tmp)
Printf : 只可以打印出格式化的字符串。只可以直接输出字符串类型的变量（不可以输出整形变量和整形 等）
fmt.Printf("%d",a)
Sprintf：格式化并返回一个字符串而不带任何输出。
s := fmt.Sprintf("a %s", "string") fmt.Printf(s)
Fprintf：来格式化并输出到 io.Writers 而不是 os.Stdout。
fmt.Fprintf(os.Stderr, “an %s\n”, “error”)
```

## Printf 格式化输出

### 通用占位符：

```
v     值的默认格式。
%+v   添加字段名(如结构体)
%#v　 相应值的Go语法表示 
%T    相应值的类型的Go语法表示 
%%    字面上的百分号，并非值的占位符
```

### 布尔值：

```
%t   true 或 false
```

### 整数值：

```
%b     二进制表示 
%c     相应Unicode码点所表示的字符 
%d     十进制表示 
%o     八进制表示 
%q     单引号围绕的字符字面值，由Go语法安全地转义 
%x     十六进制表示，字母形式为小写 a-f
%X     十六进制表示，字母形式为大写 A-F 
%U     Unicode格式：U+1234，等同于 "U+%04X"
```

### 浮点数及复数：

```
%b     无小数部分的，指数为二的幂的科学计数法，与 strconv.FormatFloat中的 'b' 转换格式一致。例如 -123456p-78 
%e     科学计数法，例如 -1234.456e+78 
%E     科学计数法，例如 -1234.456E+78 
%f     有小数点而无指数，例如 123.456 
%g     根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）输出 
%G     根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的0）输出
```

### 字符串和bytes的slice表示：

```
%s     字符串或切片的无解译字节 
%q     双引号围绕的字符串，由Go语法安全地转义 
%x     十六进制，小写字母，每字节两个字符 
%X     十六进制，大写字母，每字节两个字符
```

### 指针：

```
%p     十六进制表示，前缀 0x
```

这里没有 'u' 标记。若整数为无符号类型，他们就会被打印成无符号的。类似地，这里也不需要指定操作数的大小（int8，int64）

### 对于％ｖ来说默认的格式是：

```
bool:                    %t 
int, int8 etc.:          %d 
uint, uint8 etc.:        %d, %x if printed with %#v
float32, complex64, etc: %g
string:                  %s
chan:                    %p 
pointer:                 %p
```

由此可以看出，默认的输出格式可以使用%v进行指定，除非输出其他与默认不同的格式，否则都可以使用%v进行替代

### 对于复合对象:

里面的元素使用如下规则进行打印：

```
struct:            {field0 field1 ...} 
array, slice:      [elem0 elem1 ...] 
maps:              map[key1:value1 key2:value2] 
pointer to above:  &{}, &[], &map[]
```

### 宽度和精度：

宽度是在％之后的值，如果没有指定，则使用该值的默认值，精度是跟在宽度之后的值，如果没有指定，也是使用要打印的值的默认精度．例如：％９.２f，宽度９，精度２

```
%f:      default width, default precision 
%9f      width 9, default precision 
%.2f     default width, precision 2 
%9.2f    width 9, precision 2 
%9.f     width 9, precision 0
```

对数值而言，宽度为该数值占用区域的最小宽度；精度为小数点之后的位数。但对于 %g/%G 而言，精度为所有数字的总数。例如，对于123.45，格式 %6.2f会打印123.45，而 %.4g 会打印123.5。%e 和 %f 的默认精度为6；但对于 %g 而言，它的默认精度为确定该值所必须的最小位数。

对大多数值而言，宽度为输出的最小字符数，如果必要的话会为已格式化的形式填充空格。对字符串而言，精度为输出的最大字符数，如果必要的话会直接截断。

宽度是指"必要的最小宽度". 若结果字符串的宽度超过指定宽度时, 指定宽度就会失效。

若将宽度指定为`*'时, 将从参数中取得宽度值。

紧跟在"."后面的数串表示精度(若只有"."的话，则为".0")。若遇到整数的指示符(`d', `i', `b', `o', `x', `X', `u')的话，精度表示数值部分的长度

若遇到浮点数的指示符(`f')的话，它表示小数部分的位数。

若遇到浮点数的指示符(`e', `E', `g', `G')的话，它表示有效位数

若将精度设为`*'的话，将从参数中提取精度的值

其中对于字符串％s或者浮点类型％f,来说，精度可以截断数据的长度．如下所示．

```
func main() {
    a := 123
    fmt.Printf("%1.2d\n", a)    //123，宽度为１小于数值本身宽度，失效，而精度为２，无法截断整数
    b := 1.23
    fmt.Printf("%1.1f\n", b)    //1.2，精度为１，截断浮点型数据
    c := "asdf"
    fmt.Printf("%*.*s\n", 1, 2, c) //as，利用'*'支持宽度和精度的输入，并且字符串也可以利用精度截断
}
```

```
+     总打印数值的正负号；对于%q（%+q）保证只输出ASCII编码的字符。 
-     左对齐 
#     备用格式：为八进制添加前导 0（%#o），为十六进制添加前导 0x（%#x）或0X（%#X），为 %p（%#p）去掉前导 0x；对于 %q，若 strconv.CanBackquote 返回 true，就会打印原始（即反引号围绕的）字符串；如果是可打印字符，%U（%#U）会写出该字符的Unicode编码形式（如字符 x 会被打印成 U+0078 'x'）。 
' '  （空格）为数值中省略的正负号留出空白（% d）；以十六进制（% x, % X）打印字符串或切片时，在字节之间用空格隔开 
0     填充前导的0而非空格；对于数字，这会将填充移到正负号之后
```

对于每一个 Printf 类的函数，都有一个 Print 函数，该函数不接受任何格式化，它等价于对每一个操作数都应用 %v。另一个变参函数 Println 会在操作数之间插入空白，并在末尾追加一个换行符

不考虑占位符的话，如果操作数是接口值，就会使用其内部的具体值，而非接口本身。如下所示：

```
package main
 
import (
	"fmt"
)
 
type Sample struct{
	a   int
	str string
}
 
func main() {
	var i interface{} = Sample{1, "a"}
	fmt.Printf("%v\n", i)　　　　　　//{1 a}
}
```

### 显示参数占位符：

go中支持显示参数占位符，通过在输出格式中指定其输出的顺序即可，如下所示：

```
func main(){
    fmt.Printf("%[2]d, %[1]d\n", 11, 22)  //22, 11，先输出第二个值，再输出第一个值
}
```

## 几种输入方式的区别

```
Scan、Scanf 和 Scanln        从os.Stdin 中读取；
Fscan、Fscanf 和 Fscanln     从指定的 io.Reader 中读取；
Sscan、Sscanf 和 Sscanln     从实参字符串中读取。
Scanln、Fscanln 和 Sscanln   在换行符处停止扫描，且需要条目紧随换行符之后；
Scanf、Fscanf 和 Sscanf      需要输入换行符来匹配格式中的换行符；其它函数则将换行符视为空格。

package main
import "fmt"
import "os"
type point struct {
    x, y int
}
func main() {
    //Go 为常规 Go 值的格式化设计提供了多种打印方式。例如，这里打印了 point 结构体的一个实例。
    p := point{1, 2}
    fmt.Printf("%v\n", p) // {1 2}
    //如果值是一个结构体，%+v 的格式化输出内容将包括结构体的字段名。
    fmt.Printf("%+v\n", p) // {x:1 y:2}
    //%#v 形式则输出这个值的 Go 语法表示。例如，值的运行源代码片段。
    fmt.Printf("%#v\n", p) // main.point{x:1, y:2}
    //需要打印值的类型，使用 %T。
    fmt.Printf("%T\n", p) // main.point
    //格式化布尔值是简单的。
    fmt.Printf("%t\n", true)
    //格式化整形数有多种方式，使用 %d进行标准的十进制格式化。
    fmt.Printf("%d\n", 123)
    //这个输出二进制表示形式。
    fmt.Printf("%b\n", 14)
    //这个输出给定整数的对应字符。
    fmt.Printf("%c\n", 33)
    //%x 提供十六进制编码。
    fmt.Printf("%x\n", 456)
    //对于浮点型同样有很多的格式化选项。使用 %f 进行最基本的十进制格式化。
    fmt.Printf("%f\n", 78.9)
    //%e 和 %E 将浮点型格式化为（稍微有一点不同的）科学技科学记数法表示形式。
    fmt.Printf("%e\n", 123400000.0)
    fmt.Printf("%E\n", 123400000.0)
    //使用 %s 进行基本的字符串输出。
    fmt.Printf("%s\n", "\"string\"")
    //像 Go 源代码中那样带有双引号的输出，使用 %q。
    fmt.Printf("%q\n", "\"string\"")
    //和上面的整形数一样，%x 输出使用 base-16 编码的字符串，每个字节使用 2 个字符表示。
    fmt.Printf("%x\n", "hex this")
    //要输出一个指针的值，使用 %p。
    fmt.Printf("%p\n", &p)
    //当输出数字的时候，你将经常想要控制输出结果的宽度和精度，可以使用在 % 后面使用数字来控制输出宽度。默认结果使用右对齐并且通过空格来填充空白部分。
    fmt.Printf("|%6d|%6d|\n", 12, 345)
    //你也可以指定浮点型的输出宽度，同时也可以通过 宽度.精度 的语法来指定输出的精度。
    fmt.Printf("|%6.2f|%6.2f|\n", 1.2, 3.45)
    //要最对齐，使用 - 标志。
    fmt.Printf("|%-6.2f|%-6.2f|\n", 1.2, 3.45)
    //你也许也想控制字符串输出时的宽度，特别是要确保他们在类表格输出时的对齐。这是基本的右对齐宽度表示。
    fmt.Printf("|%6s|%6s|\n", "foo", "b")
    //要左对齐，和数字一样，使用 - 标志。
    fmt.Printf("|%-6s|%-6s|\n", "foo", "b")
    //到目前为止，我们已经看过 Printf了，它通过 os.Stdout输出格式化的字符串。Sprintf 则格式化并返回一个字符串而不带任何输出。
    s := fmt.Sprintf("a %s", "string")
    fmt.Println(s)
    //你可以使用 Fprintf 来格式化并输出到 io.Writers而不是 os.Stdout。
    fmt.Fprintf(os.Stderr, "an %s\n", "error")
}
```

## Scanf格式化输出

Scanf、Fscanf 和 Sscanf 根据格式字符串解析实参，类似于 Printf。例如，%x会将一个整数扫描为十六进制数，而 %v 则会扫描该值的默认表现格式。

格式化类似于 Printf，但也有例外，如下所示：

```
%p  没有实现
%T  没有实现
%e %E %f %F %g %G  都完全等价，且可扫描任何浮点数或复合数值
%s 和 %v     在扫描字符串时会将其中的空格作为分隔符
标记 # 和 +  没有实现
```

在输入Scanf中，宽度可以理解成输入的文本（％5s表示输入５个字符），而Scanf没有精度这种说法（没有%5.2f，只有 %5f）
