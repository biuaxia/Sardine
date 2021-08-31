---
title: 解决ZipUtil.unzip invalid CEN header (bad entry name)
date: 2021-08-31 12:00:54
toc: true
category:
 - Java
tags:
 - Java
 - 解决
 - 错误
---

7-zip压缩的zip文件里面带有中文文件名的，在其他平台上解压后中文文件名乱码。在网上查找了下资料，使用7-zip压缩zip文件，可以使用参数cu=on，强制文件名以utf-8格式编码，在其他平台上解压后正常。

下图为7-zip压缩zip格式时，文件名为中文时使用cu=on参数，强制中文文件名以utf-8编码

![d4a3c47d6a7d7b8b677741dcdbb7a435.jpg](https://b3logfile.com/file/2021/08/d4a3c47d6a7d7b8b677741dcdbb7a435-6d049e89.jpg)


<!-- more -->


via。[https://bbs.360.cn/thread-6400121-1-1.html](https://zvv.me/go/aHR0cHM6Ly9iYnMuMzYwLmNuL3RocmVhZC02NDAwMTIxLTEtMS5odG1s)

# 7-zip相关参数

## 压缩

```default
Parameter                     Default     Description
x=[0 | 1 | 3 | 5 | 7 | 9 ]    5           Sets level of compression.
m={MethodID}                  Deflate     Sets a method: Copy, Deflate, Deflate64, BZip2, LZMA, PPMd.
fb={NumFastBytes}             32          Sets number of Fast Bytes for Deflate encoder.
pass={NumPasses}              1           Sets number of Passes for Deflate encoder.
d={Size}[b|k|m]               900000      Sets Dictionary size for BZip2
mem={Size}[b|k|m]             24          Sets size of used memory for PPMd.
o={Size}                      8           Sets model order for PPMd.
mt=[off | on | {N}]           on          Sets multithreading mode.
em={EncryptionMethodID}       ZipCrypto   Sets a encryption method: ZipCrypto, AES128, AES192, AES256
tc=[off | on]                 off         Stores NTFS timestamps for files: Modification time, Creation time, Last access time.
cl=[off | on]                 off         7-Zip always uses local code page for file names.
cu=[off | on]                 off         7-Zip uses UTF-8 for file names that contain non-ASCII symbols.
```

默认情况下（如果未指定cl和cu开关），7-Zip仅对包含本地代码页不支持的符号的文件名使用UTF-8编码。

### x = [0 | 1 | 3 | 5 | 7 | 9]

设置压缩级别。 x = 0表示复制模式（无压缩）。

Deflate/Deflate64设置：

```default
Level NumFastBytes    NumPasses   Description
1     32              1           Fastest
3                                 Fast
5                                 Normal
7     64              3           Maximum
9     128             10          Ultra
```

x = 1且x = 3，Deflate方法设置快速模式进行压缩。

BZip2设置：

```default
Level Dictionary  NumPasses   Description
1     100000      1           Fastest
3     500000                  Fast
5     900000                  Normal
7                 2           Maximum
9                 7           Ultra
```

### fB = {}单词大小

设置Deflate/Deflate64编码器的快速字节数。它可以在3到258的范围内（Deflate64为257）。通常，大数字会提供更好的压缩比和更慢的压缩过程。较大的快速字节参数可以显着提高包含长相同字节序列的文件的压缩率。

### 通过=

设置Deflate编码器的通过次数。对于Deflate，它可以在1到15之间，对于BZip2，它可以在1到10之间。通常，大数字会提供更好的压缩比和更慢的压缩过程。

### d = {}尺寸并[b | K | M]

设置BZip2的字典大小。您必须以字节，千字节或兆字节为单位指定大小。字典大小的最大值是900000b。如果未指定set [b | k | m]中的任何符号，则字典大小将计算为DictionarySize = 2 ^ Size字节。

### mEM = {}尺寸并[b | K | M]

设置用于PPMd的内存大小。您必须以字节，千字节或兆字节为单位指定大小。最大值为256 MB = 2 ^ 28字节。默认值为24（16MB）。如果未指定集合[b | k | m]中的任何符号，则内存大小将计算为（2 ^ Size）字节。 PPMd使用相同数量的内存进行压缩和解压缩。

### o = {}尺寸

设置PPMd的模型顺序。大小必须在[2,16]范围内。默认值为8。

### mt = [off |在|]

设置多线程模式。如果您有多处理器或多核系统，则可以使用此开关提高速度。此选项仅影响压缩（使用任何方法）和BZip2流的解压缩。多线程模式中的每个线程使用32 MB的RAM进行缓冲。如果指定{N}，则7-Zip尝试使用N个线程。
