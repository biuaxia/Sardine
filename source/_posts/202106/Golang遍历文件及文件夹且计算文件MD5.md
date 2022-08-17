title: Golang遍历文件及文件夹且计算文件MD5
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

> 2022年6月30日14点06更新代码

```
package main

import (
	"bufio"
	"crypto/md5"
	"fmt"
	"io"
	"os"
	"path/filepath"
	"sync"
	"time"
)

/*
1. 读取文件列表
2. 并发计算文件md5
3. 等待计算完毕将数据打印
*/
func main() {
	startTime := time.Now()

	var wg sync.WaitGroup

	workers, err := readDirAllFiles("C:\\Users\\Administrator\\Downloads\\SM\\2022\\01\\01", &wg)
	// workers, err := readDirAllFiles("C:\\ProgramData\\Microsoft\\Windows\\Start Menu\\Programs\\Windsys 工具箱", &wg)
	// workers, err := readDirAllFiles("C:\\Users\\Administrator\\Downloads\\.accelerate", &wg)
	// workers, err := readDirAllFiles("C:\\Users\\Administrator\\Downloads", &wg)
	if err != nil {
		panic(err)
	}

	var result []FileInfo
	for _, v := range workers {
		result = append(result, <-v.out)
	}

	fmt.Println(len(result), "总耗时", time.Since(startTime))

	wg.Wait()
}

func createWorker(id int, wg *sync.WaitGroup) worker {
	w := worker{
		in:   make(chan FileReceiveInfo),
		out:  make(chan FileInfo),
		done: wg.Done,
	}
	go doWork(id, w)
	return w
}

func doWork(id int, w worker) {
	defer w.done()
	receiveInfo := <-w.in
	fmt.Printf("No.%-7d worker receive path: %v, size: %v\n", id, receiveInfo.FilePath, receiveInfo.FileSize)

	startMd5CalcTime := time.Now()
	fileMd5, err := MD5sum(receiveInfo.FilePath)
	fmt.Printf("No.%-7d worker time: %v\n", id, time.Since(startMd5CalcTime))
	if err != nil {
		fmt.Printf("file [%s] has error: %v\n", receiveInfo.FilePath, err)
	}
	w.out <- FileInfo{
		FilePath: receiveInfo.FilePath,
		FileMd5:  fileMd5,
	}
}

type worker struct {
	in   chan FileReceiveInfo
	out  chan FileInfo
	done func()
}

type FileReceiveInfo struct {
	FilePath string
	FileSize int64
}

type FileInfo struct {
	FilePath, FileMd5 string
}

// 读取指定目录下的所有文件
func readDirAllFiles(rootPath string, wg *sync.WaitGroup) ([]worker, error) {
	var workers []worker
	dirEntries, err := os.ReadDir(rootPath)
	if err != nil {
		return nil, err
	}

	wg.Add(len(dirEntries))

	for i, dirEntry := range dirEntries {
		if !dirEntry.IsDir() {
			worker := createWorker(i, wg)
			workers = append(workers, worker)
			entryPath := filepath.Join(rootPath, dirEntry.Name())
			fileInfo, _ := dirEntry.Info()
			worker.in <- FileReceiveInfo{
				FilePath: entryPath,
				FileSize: fileInfo.Size(),
			}
		}
	}

	return workers, nil
}

const bufferSize = 65536

// MD5sum returns MD5 checksum of filename
func MD5sum(filename string) (string, error) {
	if info, err := os.Stat(filename); err != nil {
		return "", err
	} else if info.IsDir() {
		return "", nil
	}

	file, err := os.Open(filename)
	if err != nil {
		return "", err
	}
	defer file.Close()

	hash := md5.New()
	for buf, reader := make([]byte, bufferSize), bufio.NewReader(file); ; {
		n, err := reader.Read(buf)
		if err != nil {
			if err == io.EOF {
				break
			}
			return "", err
		}

		hash.Write(buf[:n])
	}

	checksum := fmt.Sprintf("%x", hash.Sum(nil))
	return checksum, nil
}
```

---

> 2022年6月28日11点28分更新代码
>
> Fix：程序遇到无权限或其他错误时会直接退出，改为输出错误，处理下一个文件/文件夹

```
package main

import (
	"crypto/md5"
	"encoding/json"
	"errors"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"sort"
	"sync"
	"time"
)

var (
	rootDir = "C:\\Users\\Administrator\\Downloads\\SM\\2016\\03\\19"
	// rootDir = "C:\\Users\\Administrator\\Downloads"
)

/*
Ref: [Go语言并发模型：以并行处理MD5为例 - SegmentFault 思否](https://segmentfault.com/a/1190000006670880)
*/
func main() {
	startTime := time.Now()

	rootPath := os.Args[1]

	fmt.Println("计算目录", rootPath)

	// 等待动画
	go spinner(250 * time.Millisecond)

	// 计算特定目录下所有文件的 md5值
	// 然后按照路径名顺序打印结果
	m, err := MD5All(rootPath)
	if err != nil {
		fmt.Printf("\r%v\n", err)
		return
	}
	var paths []string
	for path := range m {
		paths = append(paths, path)
	}
	sort.Strings(paths)

	result := make(map[string]string, len(paths))

	for _, path := range paths {
		// fmt.Printf("%x  %s\n", m[path], path)
		result[path] = fmt.Sprintf("%x", m[path])
	}

	fmt.Println("\r总耗时", time.Since(startTime), "总文件", len(result))

	marshal, _ := json.MarshalIndent(result, "", "    ")
	writeRelFile(rootPath, marshal)
}

func writeRelFile(rootPath string, marshal []byte) {
	writeFilePath := filepath.Join(rootPath, "marshal.json")
	err := ioutil.WriteFile(writeFilePath, marshal, 0755)
	if err != nil {
		dir, errwd := os.Getwd()
		if errwd != nil {
			fmt.Printf("\r获取程序所在目录出错：%v\n", errwd)
			return
		}
		if dir == rootPath {
			fmt.Println("\r当前文件所在目录无权限，请切换目录后执行")
			return
		}
		fmt.Printf("\r写入映射文件出错：%v，尝试写入到程序所在目录：%s\n", err, dir)
		writeRelFile(dir, marshal)
		return
	}
	fmt.Printf("\r映射文件已保存至【%s】目录。\n", writeFilePath)
}

func spinner(delay time.Duration) {
	for {
		for _, r := range `-\|/` {
			fmt.Printf("\r%c", r)
			time.Sleep(delay)
		}
	}
}

/*并发版 Start*/
type result struct {
	path string
	sum  [md5.Size]byte
	err  error
}

func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
	// 对于每一个普通文件，启动一个 gorotuine 计算文件 md5 值，
	// 然后 将结果发送到 c。
	// walk 的错误结果发送到 errc。
	c := make(chan result)
	errc := make(chan error, 1)
	go func() {
		var wg sync.WaitGroup
		err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
			if err != nil {
				fmt.Printf("\rwalk has panic: %v\n", err)
				return filepath.SkipDir
			}
			if !info.Mode().IsRegular() {
				return nil
			}
			wg.Add(1)
			go func() {
				data, err := ioutil.ReadFile(path)
				select {
				case c <- result{path, md5.Sum(data), err}:
				case <-done:
				}
				wg.Done()
			}()
			// done channel 关闭时，终止 walk 函数
			select {
			case <-done:
				return errors.New("walk canceled")
			default:
				return nil
			}
		})
		// Walk 函数已经返回，所以 所有对 wg.Add 的调用都会结束
		// 启动一个 goroutine， 它会在所有发送都结束时，关闭 c。
		go func() {
			wg.Wait()
			close(c)
		}()
		// 这里不需要 select 语句，应为 errc 是缓冲管道
		errc <- err
	}()
	return c, errc
}

func MD5All(root string) (map[string][md5.Size]byte, error) {
	// MD5All 在函数返回时关闭 done channel
	// 在从 c 和 errc 接收数据前，也可能关闭
	done := make(chan struct{})
	defer close(done)

	c, errc := sumFiles(done, root)

	m := make(map[string][md5.Size]byte)
	for r := range c {
		if r.err != nil {
			return nil, fmt.Errorf("code in 138 has panic: %v", r.err)
		}
		m[r.path] = r.sum
	}
	if err := <-errc; err != nil {
		return nil, fmt.Errorf("code in 143 has panic: %v", err)
	}
	return m, nil
}

/*并发版 End*/

// ----------------------------------------- 分割线 -----------------------------------------

/*非并发版 Start*/
// MD5All 读取 root 目录下的所有文件，返回一个map
// 该 map 存储了 文件路径到文件内容 md5值的映射
// 如果 Walk 执行失败，或者 ioutil.ReadFile 读取失败，
// MD5All 都会返回错误
func MD5AllOld(root string) (map[string][md5.Size]byte, error) {
	m := make(map[string][md5.Size]byte)
	err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}
		if !info.Mode().IsRegular() {
			return nil
		}
		data, err := ioutil.ReadFile(path)
		if err != nil {
			return err
		}
		m[path] = md5.Sum(data)
		return nil
	})
	if err != nil {
		return nil, err
	}
	return m, nil
}

/*非并发版 End*/
```

---

> 2022年6月27日10点09分更新代码

```
package main

import (
	"crypto/md5"
	"encoding/json"
	"errors"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"sort"
	"sync"
	"time"
)

var (
	// rootDir = "C:\\Users\\Administrator\\Downloads\\SM\\2016\\03\\19"
	rootDir = "C:\\Users\\Administrator\\Downloads"
)

/*
Ref: [Go语言并发模型：以并行处理MD5为例 - SegmentFault 思否](https://segmentfault.com/a/1190000006670880)
*/
func main() {
	startTime := time.Now()

	// 等待动画
	go spinner(100 * time.Millisecond)

	// 计算特定目录下所有文件的 md5值
	// 然后按照路径名顺序打印结果
	m, err := MD5All(rootDir)
	if err != nil {
		fmt.Println(err)
		return
	}
	var paths []string
	for path := range m {
		paths = append(paths, path)
	}
	sort.Strings(paths)

	result := make(map[string]string, len(paths))

	for _, path := range paths {
		// fmt.Printf("%x  %s\n", m[path], path)
		result[path] = fmt.Sprintf("%x", m[path])
	}

	fmt.Println("\r总耗时", time.Now().Sub(startTime))

	marshal, _ := json.MarshalIndent(result, "", "    ")
	writeFilePath := filepath.Join(rootDir, "marshal.json")
	err = ioutil.WriteFile(writeFilePath, marshal, 0755)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("\r映射文件已保存至【%s】目录。\n", writeFilePath)
}

func spinner(delay time.Duration) {
	for {
		for _, r := range `-\|/` {
			fmt.Printf("\r%c", r)
			time.Sleep(delay)
		}
	}
}

/*并发版 Start*/
type result struct {
	path string
	sum  [md5.Size]byte
	err  error
}

func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
	// 对于每一个普通文件，启动一个 gorotuine 计算文件 md5 值，
	// 然后 将结果发送到 c。
	// walk 的错误结果发送到 errc。
	c := make(chan result)
	errc := make(chan error, 1)
	go func() {
		var wg sync.WaitGroup
		err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
			if err != nil {
				return err
			}
			if !info.Mode().IsRegular() {
				return nil
			}
			wg.Add(1)
			go func() {
				data, err := ioutil.ReadFile(path)
				select {
				case c <- result{path, md5.Sum(data), err}:
				case <-done:
				}
				wg.Done()
			}()
			// done channel 关闭时，终止 walk 函数
			select {
			case <-done:
				return errors.New("walk canceled")
			default:
				return nil
			}
		})
		// Walk 函数已经返回，所以 所有对 wg.Add 的调用都会结束
		// 启动一个 goroutine， 它会在所有发送都结束时，关闭 c。
		go func() {
			wg.Wait()
			close(c)
		}()
		// 这里不需要 select 语句，应为 errc 是缓冲管道
		errc <- err
	}()
	return c, errc
}

func MD5All(root string) (map[string][md5.Size]byte, error) {
	// MD5All 在函数返回时关闭 done channel
	// 在从 c 和 errc 接收数据前，也可能关闭
	done := make(chan struct{})
	defer close(done)

	c, errc := sumFiles(done, root)

	m := make(map[string][md5.Size]byte)
	for r := range c {
		if r.err != nil {
			return nil, r.err
		}
		m[r.path] = r.sum
	}
	if err := <-errc; err != nil {
		return nil, err
	}
	return m, nil
}

/*并发版 End*/

// ----------------------------------------- 分割线 -----------------------------------------

/*非并发版 Start*/
// MD5All 读取 root 目录下的所有文件，返回一个map
// 该 map 存储了 文件路径到文件内容 md5值的映射
// 如果 Walk 执行失败，或者 ioutil.ReadFile 读取失败，
// MD5All 都会返回错误
func MD5AllOld(root string) (map[string][md5.Size]byte, error) {
	m := make(map[string][md5.Size]byte)
	err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}
		if !info.Mode().IsRegular() {
			return nil
		}
		data, err := ioutil.ReadFile(path)
		if err != nil {
			return err
		}
		m[path] = md5.Sum(data)
		return nil
	})
	if err != nil {
		return nil, err
	}
	return m, nil
}

/*非并发版 End*/
```

---

> 2021年6月29日11点55分更新代码
>
> 运行后将会在当前目录生成文件："当前目录名.txt" 的文件

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