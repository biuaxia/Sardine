title: Channel 通道
date: 2022-06-29 14:39:00
toc: false
index_img: http://api.btstu.cn/sjbz/?lx=m_dongman&cid=7
category:
- Go
tags:
- 解决
- 次数
- 指针
---

> 切记：不要通过共享内存通信，通过通信来共享内存

- 所有channel的发送操作都是阻塞的（又叫 block）
- 使用 waiteGroup 一定要配合指针使用
- 解决实际问题一定要基于并发不能保证有序或无序的情况去排查

直接上代码，注意看注释：

```
package main

import "fmt"

type worker struct {
	in   chan int
	done chan bool
}

func main() {
	const count = 4

	var workers [count]worker
	for i := 0; i < count; i++ {
		workers[i] = createWorker(i)
	}

	for i, worker := range workers {
		worker.in <- 'a' + i
	}

	for i, worker := range workers {
		worker.in <- 'A' + i
	}

	for _, worker := range workers {
		<-worker.done
		<-worker.done
	}
}

func doWork(id int, w worker) {
	for {
		fmt.Printf("No.%d worker received: %c\n", id, <-w.in)
		go func() { w.done <- true }() // 还可以建立有缓冲的通道来解决，这里就和 done: make(chan bool, 1) 是一样的效果，不再阻塞在这里，结果无序
		// w.done <- true
	}
}

func createWorker(id int) worker {
	w := worker{
		in:   make(chan int),  // 加在 in 通道可以让两次数据都发送完，开启了两次 done 的接收，结果无序
		done: make(chan bool), // 加在 done 通道可以让第一次数据发送完，就直接开启第二次数据发送，结果无序
	}
	go doWork(id, w)
	return w
}
```

---

使用 `sync.WaitGroup` 实现：

```
package main

import (
	"fmt"
	"sync"
)

type worker struct {
	in   chan int
	done *sync.WaitGroup
}

func main() {
	const count = 4

	var wg sync.WaitGroup

	var workers [count]worker
	for i := 0; i < count; i++ {
		workers[i] = createWorker(i, &wg)
	}

	wg.Add(count * 2)
	for i, worker := range workers {
		worker.in <- 'a' + i
	}

	for i, worker := range workers {
		worker.in <- 'A' + i
	}

	wg.Wait()
}

func doWork(id int, w worker) {
	for {
		fmt.Printf("No.%d worker received: %c\n", id, <-w.in)
		w.done.Done()
	}
}

func createWorker(id int, wg *sync.WaitGroup) worker {
	w := worker{
		in:   make(chan int),
		done: wg,
	}
	go doWork(id, w)
	return w
}
```

---

将函数与 `WaitGroup` 一起使用：

```
package main

import (
	"fmt"
	"sync"
)

type worker struct {
	in   chan int
	done func()
}

func main() {
	const count = 4

	var wg sync.WaitGroup

	var workers [count]worker
	for i := 0; i < count; i++ {
		workers[i] = createWorker(i, &wg)
	}

	wg.Add(count * 2)
	for i, worker := range workers {
		worker.in <- 'a' + i
	}

	for i, worker := range workers {
		worker.in <- 'A' + i
	}

	wg.Wait()
}

func doWork(id int, w worker) {
	for {
		fmt.Printf("No.%d worker received: %c\n", id, <-w.in)
		w.done()
	}
}

func createWorker(id int, wg *sync.WaitGroup) worker {
	w := worker{
		in:   make(chan int),
		done: wg.Done,
	}
	go doWork(id, w)
	return w
}
```
