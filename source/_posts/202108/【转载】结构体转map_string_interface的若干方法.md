title: '【转载】结构体转map[string]interface{}的若干方法'
date: 2021-08-09 16:34:33
comment: false
toc: true
category:
 - Golang
tags: 
 - 转载
 - Go
 - Map
 - Struct
 - String
 - Interface
 - 方法
---

本文转载自：[结构体转map[string]interface{}的若干方法 | 李文周的博客](https://www.liwenzhou.com/posts/Go/struct2map/)

---

本文介绍了Go语言中将结构体转成`map[string]interface{}`时你需要了解的“坑”，也有你需要知道的若干方法。

我们在Go语言中通常使用结构体来保存我们的数据，例如要存储用户信息，我们可能会定义如下结构体：

```go
// UserInfo 用户信息
type UserInfo struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

u1 := UserInfo{Name: "q1mi", Age: 18}
```

假设现在要将上面的`u1`转换成`map[string]interface{}`，该如何操作呢？

## 结构体转map[string]interface{}

### JSON序列化方式

这不是很简单吗？我用JSON序列化一下u1，再反序列化成map不就可以了么。说干就干，代码如下：

```go
func main() {
	u1 := UserInfo{Name: "q1mi", Age: 18}

	b, _ := json.Marshal(&u1)
	var m map[string]interface{}
	_ = json.Unmarshal(b, &m)
	for k, v := range m{
		fmt.Printf("key:%v value:%v\n", k, v)
	}
}
```

输出：

```bash
key:name value:q1mi
key:age value:18
```

看起来没什么问题，但其实这里是有一个“坑”的。那就是Go语言中的`json`包在序列化空接口存放的数字类型（整型、浮点型等）都会序列化成float64类型。

也就是上面例子中`m["age"]`现在底层是一个`float64`了，不是个`int`了。我们来验证下：

```go
func main() {
	u1 := UserInfo{Name: "q1mi", Age: 18}

	b, _ := json.Marshal(&u1)
	var m map[string]interface{}
	_ = json.Unmarshal(b, &m)
	for k, v := range m{
		fmt.Printf("key:%v value:%v value type:%T\n", k, v, v)
	}
}
```

输出：

```bash
key:name value:q1mi value type:string
key:age value:18 value type:float64
```

很显然，出现了一个意料之外的行为，我们需要想办法规避掉。

### 反射

没办法就需要自己动手去实现了。这里使用反射遍历结构体字段的方式生成map，具体代码如下：

```go
// ToMap 结构体转为Map[string]interface{}
func ToMap(in interface{}, tagName string) (map[string]interface{}, error){
	out := make(map[string]interface{})

	v := reflect.ValueOf(in)
	if v.Kind() == reflect.Ptr {
		v = v.Elem()
	}

	if v.Kind() != reflect.Struct {  // 非结构体返回错误提示
		return nil, fmt.Errorf("ToMap only accepts struct or struct pointer; got %T", v)
	}

	t := v.Type()
	// 遍历结构体字段
	// 指定tagName值为map中key;字段值为map中value
	for i := 0; i < v.NumField(); i++ {
		fi := t.Field(i)
		if tagValue := fi.Tag.Get(tagName); tagValue != "" {
			out[tagValue] = v.Field(i).Interface()
		}
	}
	return out, nil
}
```

验证一下：

```go
m2, _ := ToMap(&u1, "json")
for k, v := range m2{
	fmt.Printf("key:%v value:%v value type:%T\n", k, v, v)
}
```

输出：

```bash
key:name value:q1mi value type:string
key:age value:18 value type:int
```

这一次`map["age"]`的类型就对了的。

### 第三方库structs

除了自己实现，Github上也有现成的轮子，例如第三方库：[https://github.com/fatih/structs](https://github.com/fatih/structs)。

它使用的自定义结构体tag是`structs`:

```go
// UserInfo 用户信息
type UserInfo struct {
	Name string `json:"name" structs:"name"`
	Age  int    `json:"age" structs:"age"`
}
```

用法很简单：

```go
m3 := structs.Map(&u1)
for k, v := range m3 {
	fmt.Printf("key:%v value:%v value type:%T\n", k, v, v)
}
```

`structs`这个包也有很多其他的使用示例，大家可以去查看文档。但是需要注意的是目前这个库已经被作者设置为只读了。

## 嵌套结构体转map[string]interface{}

`structs`本身是支持嵌套结构体转`map[string]interface{}`的，遇到结构体嵌套它会转换为`map[string]interface{}`嵌套`map[string]interface{}`的模式。

我们定义一组嵌套的结构体如下：

```go
// UserInfo 用户信息
type UserInfo struct {
	Name string `json:"name" structs:"name"`
	Age  int    `json:"age" structs:"age"`
	Profile `json:"profile" structs:"profile"`
}

// Profile 配置信息
type Profile struct {
	Hobby string `json:"hobby" structs:"hobby"`
}
```

声明结构体变量u1：

```go
u1 := UserInfo{Name: "q1mi", Age: 18, Profile: Profile{"双色球"}}
```

### 第三方库structs

代码和上面的其实是一样的：

```go
m3 := structs.Map(&u1)
for k, v := range m3 {
	fmt.Printf("key:%v value:%v value type:%T\n", k, v, v)
}
```

输出结果：

```bash
key:name value:q1mi value type:string
key:age value:18 value type:int
key:profile value:map[hobby:双色球] value type:map[string]interface {}
```

从结果来看最后嵌套字段`profile`是`map[string]interface {}`，属于map嵌套map。

### 使用反射转成单层map

如果我们想把嵌套的结构体转换成一个单层map该怎么做呢？

我们把上面反射的代码稍微修改一下就可以了：

```go
// ToMap2 将结构体转为单层map
func ToMap2(in interface{}, tag string) (map[string]interface{}, error) {

	// 当前函数只接收struct类型
	v := reflect.ValueOf(in)
	if v.Kind() == reflect.Ptr { // 结构体指针
		v = v.Elem()
	}
	if v.Kind() != reflect.Struct {
		return nil, fmt.Errorf("ToMap only accepts struct or struct pointer; got %T", v)
	}

	out := make(map[string]interface{})
	queue := make([]interface{}, 0, 1)
	queue = append(queue, in)

	for len(queue) > 0 {
		v := reflect.ValueOf(queue[0])
		if v.Kind() == reflect.Ptr { // 结构体指针
			v = v.Elem()
		}
		queue = queue[1:]
		t := v.Type()
		for i := 0; i < v.NumField(); i++ {
			vi := v.Field(i)
			if vi.Kind() == reflect.Ptr { // 内嵌指针
				vi = vi.Elem()
				if vi.Kind() == reflect.Struct { // 结构体
					queue = append(queue, vi.Interface())
				} else {
					ti := t.Field(i)
					if tagValue := ti.Tag.Get(tag); tagValue != "" {
						// 存入map
						out[tagValue] = vi.Interface()
					}
				}
				break
			}
			if vi.Kind() == reflect.Struct { // 内嵌结构体
				queue = append(queue, vi.Interface())
				break
			}
			// 一般字段
			ti := t.Field(i)
			if tagValue := ti.Tag.Get(tag); tagValue != "" {
				// 存入map
				out[tagValue] = vi.Interface()
			}
		}
	}
	return out, nil
}
```

测试一下：

```go
m4, _ := ToMap2(&u1, "json")
for k, v := range m4 {
	fmt.Printf("key:%v value:%v value type:%T\n", k, v, v)
}
```

输出：

```go
key:name value:q1mi value type:string
key:age value:18 value type:int
key:hobby value:双色球 value type:string
```

这下我们就把嵌套的结构体转为单层的map了，但是要注意这种场景下结构体和嵌套结构体的字段就需要避免重复。

