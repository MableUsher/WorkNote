### 数组
1. 数组的长度以出现的最大的索引为准，没有明确初始化的元素依然用0值初始化
    *  var d = [...]int{1, 2, 4: 5, 6} // 定义长度为6的int型数组, 元素为 1, 2, 0, 0, 5, 6
2. for range迭代数组
```
for key, value := range collection {     }

```



### 切片
1. Cap成员表示切片指向的内存空间的最大容量（对应元素的个数，而不是字节数）
2. append()
3. copy()
4. 使用 copy 函数要注意对于 copy(dst, src)，要初始化 dst 的 size，否则无法复制

### channel
1. 对于带缓冲的Channel，对于Channel的第K个接收完成操作发生在第K+C个发送操作完成之前，其中C是Channel的缓存大小
2. 通道遵循先进先出原则
### 指针
1. &和指针的区别 https://www.cnblogs.com/wpclw/p/8405308.html
2. 怎样区分&是引用还是取地址符呢？方法是：判断&a这样的形式前是否有类型符即int &a=b;如果有类型符(int)则是引用，否则是取地址运算符。
3. 表达式new(T)将创建一个T类型的匿名变量，初始化为T类型的零值，然后返回变量地址，返回的指针类型为 星号T
4.  cla := new(Class)   //这个new方法，就相当于  cla := &Class{}，是一个取地址的操作。

### struct（结构化类型）
### goroutine
1. 启动一个新的协程时，协程的调用会立即返回。与函数不同，程序控制不会去等待 Go 协程执行完毕。在调用 Go 协程之后，程序控制会立即返回到代码的下一行，忽略该协程的任何返回值。
2. 如果希望运行其他 Go 协程，Go 主协程必须继续运行着。如果 Go 主协程终止，则程序终止，于是其他 Go 协程也不会继续运行。
### sync.Mutex互斥锁
1. 已经锁定的 Mutex 并不与特定的 goroutine 相关联，这样可以利用一个 goroutine 对其加锁，再利用其他 goroutine 对其解锁
2. 每次一个goroutine访问bank变量时(这里只有balance余额变量)，它都会调用mutex的Lock方法来获取一个互斥锁。如果其它的goroutine已经获得了这个锁的话，这个操作会被阻塞直到其它goroutine调用了Unlock使该锁变回可用状态。https://docs.hacknode.org/gopl-zh/ch9/ch9-02.html
3. 临界区 在Lock和Unlock之间的代码段中的内容goroutine可以随便读取或者修改，这个代码段叫做临界区

### defer（延迟函数）
1. defer后面的函数在defer语句所在的函数执行结束的时候会被调用
2. 若想在函数结束之前就执行defer，可用匿名函数包裹defer

### 回调函数
1. 我们先定义了主函数和回调函数，然后再去调用主函数，将回调函数传进去。　　定义主函数的时候，我们让代码先去执行callback()回调函数，但输出结果却是后输出回调函数的内容。这就说明了主函数不用等待回调函数执行完，可以接着执行自己的代码。所以一般回调函数都用在耗时操作上面。比如ajax请求，比如处理文件等。

### WaitGroup
1. 在创建协程的时候同时给wg计数，在携程中执行完后给wg减数，最后在main中使用wg.wait()，等待wg归零，即所有的协程都执行完毕

### string包
1. strings.split()
    * func Split(s, sep string) []string{} 按照sep规则进行切割s字符串，返回值为切片类型

### select
1. 一个select语句用来选择哪个case中的发送或接收操作可以被立即执行
2. 如果有一个或多个IO操作可以完成，则Go运行时系统会随机的选择一个执行，否则的话，如果有default分支，则执行default分支语句，如果连default都没有，则select语句会一直阻塞，直到至少有一个IO操作可以进行

### note
1. golang中实现协程间通讯有两种，
    1. 共享内存型，使用全局变量+mutex锁来实现数据共享
    2. 消息传递型，使用channel机制进行异步通讯


### recover
* golang中的宕机恢复机制，类似于异常捕获机制try/catch
* 使用：recover仅在defer函数中有效，如果当前的协程发生panic，recover会使程序从panic中恢复，并返回panic信息，导致发生panic的函数不会继续运行，但是可以正常返回，整个程序从宕机点退出当前函数后继续执行
* 在未发生panic时调用recover，会返回nil

### sync.Once
* 对一个sync.Once类型值的指针方法Do的有效调用次数永远会是1。
* 