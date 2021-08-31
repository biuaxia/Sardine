title: 【转载】Go语言实现Base64、Base58编码与解码
date: 2021-07-26 09:51:49
toc: true
category:
 - Golang
tags: 
 - Go
 - Base64
 - 解码
 - 编码
---

本文转载自：[Go语言实现Base64、Base58编码与解码](https://juejin.cn/post/6988718600634761229?utm_source=gold_browser_extension)

---

## 常见的编码

base64:26个小写字母、26个大写字母、10个数字、/、+

base58(区块链)：去掉6个容易混淆的，去掉0，大写的O、大写的I、小写的L、/、+/、+影响双击选择


<!-- more -->


## go语言实现base64的编码与解码

### Base64编码原理

### 实现Base64的编码与解码

base64

```go
package main

import (
	"encoding/base64"
	"fmt"
)

func Base64Encoding(str string) string {  //Base64编码
	src := []byte(str)
	res := base64.StdEncoding.EncodeToString(src)  //将编码变成字符串
	return res
}

func Base64Decoding(str string) string {  //Base64解码
	res,_:=base64.StdEncoding.DecodeString(str)
	return string(res)
}

func main() {
	src := "FanOne"
	res := Base64Encoding(src) // 编码
	fmt.Println("FanOne 编码成为：",res)
	ret := Base64Decoding(res)  //解码
	fmt.Println(res,"解码成为：",ret)
}
```

结果为

```shell
FanOne 编码成为： RmFuT25l
RmFuT25l 解码称为： FanOne
```

## go语言实现base58编码与解码

Base58编码表

![](https://b3logfile.com/file/2021/07/solo-fetchupload-2944112033814691389-fe687dad.png)

### base58的编码过程

![](https://b3logfile.com/file/2021/07/solo-fetchupload-2674335680101247606-32236a65.png)

1. 将字符串的每个字节换算成ASCII，字符串实际上就是256进制的数字组合
2. 将256进制的数字转换成10进制数字
3. 将10进制数字转换成58进制数字（除以58，每次得到的余数就是对应的58进制，0用编码1来代表）
4. 将58进制数字对照58编码表找到对应的字符

例子：

1. Fan对应的换成ASCII：77 97 110
2. 将77 97 110的256转换10进值：77*256* 256 + 97*256 + 110 = 4612462
3. 将4612462除以58，每次得到的余数就是对应的58进制，0用编码1来代表：
4. 用余数的值去查表

### base58编解码具体实现

```go
package main

import (
	"bytes"
	"fmt"
	"math/big"
)


var base58= []byte("123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz")

func Base58Encoding(str string) string { 		//Base58编码
	//1. 转换成ascii码对应的值
	strByte := []byte(str)
	//fmt.Println(strByte) // 结果[70 97 110]
	//2. 转换十进制
	strTen := big.NewInt(0).SetBytes(strByte)
	//fmt.Println(strTen)  // 结果4612462
	//3. 取出余数
	var modSlice []byte
	for strTen.Cmp(big.NewInt(0)) > 0 {
		mod:=big.NewInt(0)  			//余数
		strTen58:=big.NewInt(58)
		strTen.DivMod(strTen,strTen58,mod)  //取余运算
		modSlice = append(modSlice, base58[mod.Int64()])    //存储余数,并将对应值放入其中
 	}
	//  处理0就是1的情况 0使用字节'1'代替
	for _,elem := range strByte{
		if elem!=0{
			break
		}else if elem == 0{
			modSlice = append(modSlice,byte('1'))
		}
	}
	//fmt.Println(modSlice)   //结果 [12 7 37 23] 但是要进行反转，因为求余的时候是相反的。
	//fmt.Println(string(modSlice))  //结果D8eQ
	ReverseModSlice:=ReverseByteArr(modSlice)
	//fmt.Println(ReverseModSlice)  //反转[81 101 56 68]
	//fmt.Println(string(ReverseModSlice))  //结果Qe8D
	return string(ReverseModSlice)
}

func ReverseByteArr(bytes []byte) []byte{  	//将字节的数组反转
	for i:=0; i<len(bytes)/2 ;i++{
		bytes[i],bytes[len(bytes)-1-i] = bytes[len(bytes)-1-i],bytes[i]  //前后交换
	}
	return bytes
}

//就是编码的逆过程
func Base58Decoding(str string) string { //Base58解码
	strByte := []byte(str)
	//fmt.Println(strByte)  //[81 101 56 68]
	ret := big.NewInt(0)
	for _,byteElem := range strByte{
		index := bytes.IndexByte(base58,byteElem) //获取base58对应数组的下标
		ret.Mul(ret,big.NewInt(58))  			//相乘回去
		ret.Add(ret,big.NewInt(int64(index)))  //相加
	}
	//fmt.Println(ret) 	// 拿到了十进制 4612462
	//fmt.Println(ret.Bytes())  //[70 97 110]
	//fmt.Println(string(ret.Bytes()))
	return string(ret.Bytes())
}

func main() {
	src := "Fan"
	res := Base58Encoding(src)
	fmt.Println(res)  //Qe8D
	resD:=Base58Decoding(res)
	fmt.Println(resD)  //Fan
}
```

## 最后

![](https://b3logfile.com/file/2021/07/solo-fetchupload-2433902292403564600-dc686c62.png)
