title: 【转载】Go语言中的单例模式
date: 2021-08-09 16:35:33
comment: false
toc: true
category:
 - Golang
tags: 
 - 转载
 - Go
 - 设计
 - 模式
 - 单例
 - 错误
---

本文转载自：[Go语言中的单例模式 | 李文周的博客](https://www.liwenzhou.com/posts/Go/singleton_in_go/)

---

在过去的几年中，Go语言的发展是惊人的，并且吸引了很多由其他语言（Python、PHP、Ruby）转向Go语言的跨语言学习者。 Go语言太容易实现并发了，以至于它在很多地方被不正确的使用了。

## Go语言中的单例模式

在过去的几年中，Go语言的发展是惊人的，并且吸引了很多由其他语言（Python、PHP、Ruby）转向Go语言的跨语言学习者。

在过去的很长时间里，很多开发人员和初创公司都习惯使用Python、PHP或Ruby快速开发功能强大的系统，并且大多数情况下都不需要担心内部事务如何工作，也不需要担心线程安全性和并发性。直到最近几年，多线程高并发的系统开始流行起来，我们现在不仅需要快速开发功能强大的系统，而且还要保证被开发的系统能够足够快速运行。（我们真是太难了☺️）

对于被Go语言天生支持并发的特性吸引来的跨语言学习者来说，我觉着掌握Go语言的语法并不是最难的，最难的是突破既有的思维定势，真正理解并发和使用并发来解决实际问题。

Go语言太容易实现并发了，以至于它在很多地方被不正确的使用了。

### 常见的错误

有一些错误是很常见的，比如不考虑并发安全的单例模式。就像下面的示例代码：

```go
package singleton

type singleton struct {}

var instance *singleton

func GetInstance() *singleton {
	if instance == nil {
		instance = &singleton{}   // 不是并发安全的
	}
	return instance
}
```

在上述情况下，多个goroutine可以执行第一个检查，并且它们都将创建该`singleton`类型的实例并相互覆盖。无法保证它将在此处返回哪个实例，并且对该实例的其他进一步操作可能与开发人员的期望不一致。

不好的原因是，如果有代码保留了对该单例实例的引用，则可能存在具有不同状态的该类型的多个实例，从而产生潜在的不同代码行为。这也成为调试过程中的一个噩梦，并且很难发现该错误，因为在调试时，由于运行时暂停而没有出现任何错误，这使非并发安全执行的可能性降到了最低，并且很容易隐藏开发人员的问题。

### 激进的加锁

也有很多对这种并发安全问题的糟糕解决方案。使用下面的代码确实能解决并发安全问题，但会带来其他潜在的严重问题，通过加锁把对该函数的并发调用变成了串行。

```go
var mu Sync.Mutex

func GetInstance() *singleton {
    mu.Lock()                    // 如果实例存在没有必要加锁
    defer mu.Unlock()

    if instance == nil {
        instance = &singleton{}
    }
    return instance
}
```

在上面的代码中，我们可以看到在创建单例实例之前通过引入`Sync.Mutex`和获取Lock来解决并发安全问题。问题是我们在这里执行了过多的锁定，即使我们不需要这样做，在实例已经创建的情况下，我们应该简单地返回缓存的单例实例。在高度并发的代码基础上，这可能会产生瓶颈，因为一次只有一个goroutine可以获得单例实例。

因此，这不是最佳方法。我们必须考虑其他解决方案。

### Check-Lock-Check模式

在C ++和其他语言中，确保最小程度的锁定并且仍然是并发安全的最佳和最安全的方法是在获取锁定时利用众所周知的`Check-Lock-Check`模式。该模式的伪代码表示如下。

```go
if check() {
    lock() {
        if check() {
            // 在这里执行加锁安全的代码
        }
    }
}
```

该模式背后的思想是，你应该首先进行检查，以最小化任何主动锁定，因为IF语句的开销要比加锁小。其次，我们希望等待并获取互斥锁，这样在同一时刻在那个块中只有一个执行。但是，在第一次检查和获取互斥锁之间，可能有其他goroutine获取了锁，因此，我们需要在锁的内部再次进行检查，以避免用另一个实例覆盖了实例。

如果将这种模式应用于我们的`GetInstance()`方法，我们会写出类似下面的代码：

```go
func GetInstance() *singleton {
    if instance == nil {     // 不太完美 因为这里不是完全原子的
        mu.Lock()
        defer mu.Unlock()

        if instance == nil {
            instance = &singleton{}
        }
    }
    return instance
}
```

通过使用`sync/atomic`这个包，我们可以原子化加载并设置一个标志，该标志表明我们是否已初始化实例。

```go
import "sync"
import "sync/atomic"

var initialized uint32
... // 此处省略

func GetInstance() *singleton {

    if atomic.LoadUInt32(&initialized) == 1 {  // 原子操作 
		    return instance
	  }

    mu.Lock()
    defer mu.Unlock()

    if initialized == 0 {
         instance = &singleton{}
         atomic.StoreUint32(&initialized, 1)
    }

    return instance
}
```

但是……这看起来有点繁琐了，我们其实可以通过研究Go语言和标准库如何实现goroutine同步来做得更好。

### Go语言惯用的单例模式

我们希望利用Go惯用的方式来实现这个单例模式。我们在标准库`sync`中找到了`Once`类型。它能保证某个操作仅且只执行一次。下面是来自Go标准库的源码（部分注释有删改）。

```go
// Once is an object that will perform exactly one action.
type Once struct {
	// done indicates whether the action has been performed.
	// It is first in the struct because it is used in the hot path.
	// The hot path is inlined at every call site.
	// Placing done first allows more compact instructions on some architectures (amd64/x86),
	// and fewer instructions (to calculate offset) on other architectures.
	done uint32
	m    Mutex
}

func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 0 { // check
		// Outlined slow-path to allow inlining of the fast-path.
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()                          // lock
	defer o.m.Unlock()
	
	if o.done == 0 {                    // check
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

这说明我们可以借助这个实现只执行一次某个函数/方法，`once.Do()`的用法如下：

```go
once.Do(func() {
    // 在这里执行安全的初始化
})
```

下面就是单例实现的完整代码，该实现利用`sync.Once`类型去同步对`GetInstance()`的访问，并确保我们的类型仅被初始化一次。

```go
package singleton

import (
    "sync"
)

type singleton struct {}

var instance *singleton
var once sync.Once

func GetInstance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```

因此，使用`sync.Once`包是安全地实现此目标的首选方式，类似于Objective-C和Swift（Cocoa）实现`dispatch_once`方法来执行类似的初始化。

### 结论

当涉及到并发和并行代码时，需要对代码进行更仔细的检查。始终让你的团队成员执行代码审查，因为这样的事情很容易就会被发现。

所有刚转到Go语言的新开发人员都必须真正了解并发安全性如何工作以更好地改进其代码。即使Go语言本身通过允许你在对并发性知识知之甚少的情况下设计并发代码，也完成了许多繁重的工作。在某些情况下，单纯的依靠语言特性也无能为力，你仍然需要在开发代码时应用最佳实践。

翻译自[http://marcio.io/2015/07/singleton-pattern-in-go/](http://marcio.io/2015/07/singleton-pattern-in-go/)，考虑到可读性部分内容有修改。

