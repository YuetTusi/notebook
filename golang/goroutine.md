
Go语言最大的一个优势就是天生支持并发，这是它被称为21世纪的C语言的一大原因。

并发，指是同一时间内同时执行多个任务，它与“并行”在不同之处在于：

- 并发是把任务在不同的时间点交给处理器进行处理。在同一时间点，任务并不会同时运行
- 并行是把每一个任务分配给每一个处理器独立完成。在同一时间点，任务一定是同时运行

其中区别的关键在于是否“**同时进行**”

举例子：你用你的手机与多个女朋友同时聊天，即并发（先给第一位女友回消息，再回第二位）；多个人同时和你的女朋友聊天，即并行（多人同时给女友发消息）。


Go语言执行并发称为goroutine，在一个普通函数前加上`go`关键字即开启了并发执行：

```go
go test() //并发执行test函数
```

由于将某个函数并发执行，因此函数的返回值无法拿到，即使该函数有返回值也无法取得。


```go
func say(name string) {
	fmt.Printf("%s say Hi\n", name)
}

func main() {
	go say("Tom")
	go say("Peter")

	fmt.Println("运行结束")
}
```
运行结果：

```bash
运行结束
```

以上例子，将say函数并发执行2次，参数接收两个名字分别输出消息，但是运行结果发现没有任何输出，程序运行直接结束。这是因为`go`开启的并发执行要消耗时间，在主线程main函数执行完毕时`say()`还没有结束，因此没来得及输出消息，可以通过`Sleep()`函数让`main()`函数延迟：

```go
func main() {

	go say("Tom")
	go say("Peter")
	time.Sleep(time.Second) //延迟1秒
	fmt.Println("运行结束")
}
```
结果：
```bash
Peter say Hi
Tom say Hi
运行结束
```

但是`Sleep()`函数仅仅用于上面代码的演示，它的真实作用是让CPU空闲1秒，在实际开发中不要用这种方法去等待并行任务，通常我们使用sync包下的`WaitGroup`来处理

```go
var g sync.WaitGroup

func say(name string) {

	fmt.Printf("%s say Hi\n", name)
	g.Done() //当say完成，让阻塞计数-1
}

func main() {

	go say("Tom")
	go say("Peter")
	g.Add(2) //开启2个并发任务，告知WaitGroup
	g.Wait() //阻塞main直到并发任务为0
	fmt.Println("运行结束")
}
```
运行结果：
```go
Peter say Hi
Tom say Hi
运行结束
```

WaitGroup用于控制并发任务的同步等待，当启动多个并发任务，使用Add来设置并发任务的数量，当执行完毕Done函数即将计数减1，这样就可以精确的控制主线程的等待时间。

### 通道

单纯的让函数并发执行没有任何意义，并发函数之间必然有数据的通信，而通道channel是并发任务之间通信的信道，Go语言通过CSP模型来实现通信共享内存。

首先明确channel是一种类型，它遵循<strong style="color:red;">先进先出</strong>的原则，因此数据通过channel发送是<strong style="color:red;">有序</strong>的。


#### 声明与创建

使用channel前要先进行声明，也要明确数据类型。

```go
var c1 chan string //一个字符串通道
var c2 chan []int  //一个整型切片通道
```

声明通道使用`chan`类型来定义，定义时要明确给出类型。

通道是引用类型（未分配内存时为nil），创建要使用make来分配内存：

```go
c1 := make(chan string, 1) //一个字符串通道，有1个大小缓冲区
c2 := make(chan []int)  //一个数值切片通道，无缓冲区
```

make的第1个参数指明通道类型；参数2为缓冲区大小，如果不填写则为无缓冲区通道。

通道也可以使用`len()`和`cap()`来取得数据长度和容量大小。

> 小结：`make()`函数可用于向Slice、Map和Channel分配空间（目前只用于这3种数据类型）

#### 操作

通道的操作有3个：发送（send），接收（receive），关闭（close）。

**发送**和**接收**都使用同一个符号： `<-`

```go
c1 := make(chan string, 1) //一个字符串通道
c1 <- "Hello"// 将Hello发送给c1通道
```

从c1通道中接收数据：
```go
msg := <-c1 //从c1通道中接收结果并赋值
<-c1 //也可以只接收结果
```
符号`<-`作为发送与接收的区别仅在channel的左边还是右边

关闭通道使用close函数：

```go
close(c1)
```

对已经关闭的通道操作，有如下几点要注意：

- 对一个已经关闭的通道再进行close操作，会引发宕机（panic）
- 对一个已经关闭的通道发送消息，会引发宕机（panic）
- 对一个已经关闭的通道接收消息，会一直获取直到数据为空；之后再接收消息不会引发错误
- 对一个已经关闭且无数据的通道接收消息，会得到零值

#### 缓冲区

在声明一个channel时会有缓冲区与无缓冲区的分别（`make()`函数有无第2个参数）。

如果是无缓冲区的channel，则发送给通道的数据必须由某个并发函数接收，否则会阻塞线程的执行。


以下演示了有缓冲区的通道，定义一组切片数据将数值发送给通道，交由`Square()`函数求平方后输出结果：
```go
var group sync.WaitGroup

func main() {
	defer group.Wait()

	var data = []int{2, 3, 4} //定义3个元素的切片

	c := make(chan int, len(data)) //定义有缓冲区的channel，长度为切片数据的长度3

	for _, value := range data {
	    //循环切片，发送给channel
		c <- value
	}
	close(c) //发送结束，关闭channel
	group.Add(1)
	go Square(&c) //并发运行Square()
}

//接收数据求平方值
func Square(c *chan int) {
	defer group.Done()
	for value := range *c {
	    //循环channel中的值，将每个数值求平方后输出
		fmt.Printf("%d^2 = %d\n", value, value*value)
	}
}
```

运行结果：
```bash
2^2 = 4
3^2 = 9
4^2 = 16
```

而无缓冲区的通道，必须有一个并发函数先去接收通道中的数据，否则会阻塞程序运行造成死锁（dead lock）：

```go
var group sync.WaitGroup

func main() {

	c := make(chan string) //无缓冲区

	group.Add(1)
	go ReadName(&c) //启动并发运行ReadName()，此时阻塞等待c通道发来的数据
	c <- "Tom" //向c发送Tom
	group.Wait()
}

//读取通道中姓名输出
func ReadName(c *chan string) {
	defer group.Done()
	name := <-*c
	fmt.Printf("接收到姓名：%s", name)
}
```

运行结果：

```bash
接收到姓名：Tom
```

无缓冲区的channel实际相当于接力赛跑，数据必须交由一个函数去处理，处理了数据，此时channel中的数据才会清空，否则会造成死锁。

以上示例如果将发送消息的语句提前，放到并发运行`ReadName()`函数之前，即造成死锁：

```go
func main() {

	c := make(chan string)
	c <- "Tom" //先发送数据，此时未运行ReadName()
	group.Add(1)
	go ReadName(&c)
	group.Wait()
}
```
运行失败：

```bash
fatal error: all goroutines are asleep - deadlock!
```

#### 单向通道

当定义了一个通道，此时数据即可以读取也可以接收，为双向。

一般为了让代码更加严谨且健壮，我们可以将通道定义为单向，即“仅读取”或是“仅接收”。

单向通道一般在函数的参数与返回值中使用，以避免通道操作的误用。

定义单向通道只需在chan关键字的左边或是右边加上符号`<-`即可：

```go
c1 := make(chan<- string) //只写通道
c2 := make(<-chan string) //只读通道
```

区别仅为符号`<-`在chan的左边还是右边，有了单向通道的写法，在定义函数参数可以严格的限制：

```go
func ReadName(c <-chan string) {

    c<- "Jack" //报错，参数c为只读通道
}
```

#### 小结

channel|nil|非空|空的|满了|没满
---|---|---|---|---|---
接收|阻塞|接收值|阻塞|接收值|接收值
发送|阻塞|发送值|发送值|阻塞|发送值
关闭|panic|关闭成功，读完数据返回零值|关闭成功，返回零值|关闭成功，读完数据返回零值|关闭成功，读完数据返回零值

### 锁

当并发运行的代码共同修改同一个数据时，就会牵扯到锁的运用。

举例：

```go
var group sync.WaitGroup
var count int = 0 //全局变量，计数

func main() {
	group.Add(3)
	for i := 0; i < 3; i++ {
		go add() //开启3个协程，同时操作count累加
	}
	group.Wait()
	fmt.Printf("count:%d", count)
}

//每执行一次函数，值累加1000
func add() {
	defer group.Done()
	for i := 0; i < 1000; i++ {
		count++
	}
}
```
执行上述代码，会发现每次执行的结果都不同：

```txt
第1次运行结果：count:2856
第2次运行结果：count:3000
第3次运行结果：count:2000
第4次运行结果：count:3000
第5次运行结果：count:3000
```

这是由于每一次`add()`在执行修改count变量的这一过程，并不具有原子性，因此最终的结果并不是我们想要的。

这就需要锁来帮助。

Go语言中的锁都在sync包中定义，分为**互斥锁（Mutex）**和**读写锁（RWMutex）**

声明一个互斥锁：

```go
lock := new(sync.Mutex) //lock是一个Mutex的指针类型
lock := &sync.Mutex{} //这样也可以
```

定义好了互斥锁，在更新count时使用`Lock()`加锁和`Unlock()`解锁即可：

```go
//调用时传入lock即可： add(lock)
func add(lock *sync.Mutex) {
	defer group.Done()

	for i := 0; i < 1000; i++ {
		lock.Lock() //更新count前加锁
		count++
		lock.Unlock() //解锁
	}
}
```

这样，在当多个协程同时修改count时因为有锁的加持，每修改一次count，会让其操作变为原子操作，让下一个协程等待当前锁的释放。

如此不管运行多少次，其结果都是正确的：

```bash
count:3000
```

互斥锁用法简单，但是性能稍差。这是因为锁未区分读与写的操作，而读写锁（RWMutex）则会更加详细的区分读写两种状态，提高锁的性能。

试想，当一个数据只有写的时候才会引发冲突，而读一般来说不会影响。

