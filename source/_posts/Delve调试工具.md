---
title: Delve调试工具
date: 2020-01-20 10:58:52
tags: [Go,服务端开发]
categories: Go栈
---

![](/start.jpg)

# Delve调试工具

​    为什么会选择delve呢，很多时候我们会把调试日志和运行时日志用自己能够处理的方式存储起来，比如说直接在开发阶段用log工具，在先上阶段使用日志库文件存储程序运行时的异常信息和不确定性的运行结果，或者是用kafka工具来接受导出管理日志。但是这一些都算是事后的事儿了，我们通过他们进行日志相关的查阅，像侦探一样一条一条的去发现线索。这个在一些特定情况下，比如无法重现，必须追踪调用栈，知道每个变量的运行时的值等一些列问题时，log工具显得有点手短了，而这一些delve工具可以很好的帮我们解决。

Dlv，也称为Delve，是Go语言的源码调试工具。由derekparker开发，开源在Github。在Mac上配置Go语言开发环境的时候，经常碰到的问题就是Dlv调用总是不成功，无法启动应用，无法调试等等。大部分的问题都与Mac的安全机制有关。Mac上使用codesign对应用进行签名，没有签名的程序会受到一些限制，例如无法作为调试程序。

Delve是一个第三方的工具，所以我们需要事先准备好delve工具

## Delve安装

## Delve的使用

比如我们现在有一个很简单的程序，做一个和运算。

代码结构是：

```go
Add.go  : func Add(a int,b int) int
main.go : main()-> c := Add(10,20)
```

在Add文件里面有Add函数，mian里面进行调用，此时，使用

```go
go build
```

```go
./Add
```

可以直接运行该程序，当然我们的重点在于如何使用delve来调试该程序。

具体步骤：

1. 使用dlv debug 命令调试该程序
2. 使用 [   b   ] 命令设置断点，阻塞调试运行
3. 在终端使用相关的命令行查看运行时程序的状态

如:

进入dlv调试模式，并且使用该项目作为调试对象

```bash
dlv debug github.com/daniel/Add
```

**b**  是对main包下面的main方法设置断点，			【  breakpoint  】

```
b main.main
```

在调试模式下执行此程序， **c**								【  continue  】

```
c
```

```go
	1 package main
	2 import "fmt"
	3
=>4 func main(){
	5 var a = 10
	6 var b = 20
	7 c:=Add(a,b)
	8 fmt.Println("c:",c)
	9 }
```

就会在命令行出现如上的代码运行时况，**next**是执行下一步指令

```
next
```

```go
	1 package main
	2 import "fmt"
	3
  4 func main(){
=>5 var a = 10
	6 var b = 20
	7 c:=Add(a,b)
	8 fmt.Println("c:",c)
	9 }
```

再执行 **next**指令 ，我们可以通过**p a**查看运行时**a变量**的值  【 print 】

```
p a
```

出现如下，显示a变量的值为 10

```
(div) p a
10
```

同理，当我们运行到函数Add所在行的时候，我们可以通过   **s**  命令进入该函数，查看运行状态。 【  step   】

```
s
```

显示函数内部相关的调试步骤。

**r** 命令是重新执行该程序 【 run 】

```
r
```

总结一下，dlv的常用命令：

- b 设置断点
- c 直接运行到断点处
- s 单步执行，遇到函数进入内部
- next 逐行执行 遇到函数直接运行到函数返回，就是不进入内部
- r 是重新执行该程序

到这，如果觉得dlv不过如此的话，那就大错特错了，dlv强大之处它可以跨机器调试对应的进程。

### Dlv调试正在运行的程序

首先，新建一个程序，因为需要用dlv工具attach到该程序的进程，所以这程序得是一个能够在后台长时间驻留的程序。如下:

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	for {
		var times int

		var currentTime time.Time

		time.Sleep(3 * time.Second)

		times++
		currentTime = time.Now()

		fmt.Printf("process had runed %d times,current time is : %v \n", times, currentTime)

	}

}
```

该程序可以模拟我们线上运行的程序，它每隔3秒打印一次运行次数和时间。

```
process had runed 1 times,current time is : 2018-04-07
```

在linux机器上可以通过： 

```bash
ps aux|grep dlv_attach
```

查看当前程序dlv_attack的进程情况。找到程序的进程id

使用：dlv attack 进程id  将dlv调试工具套接到当前运行的程序上

```bash
dlv attack process_id
```

此时运行的程序会进入调试模式，然后可以通过相关命令进行查看程序运行状态。

### Dlv多线程调试模式

在我们启动多个**goroutine来并发执行**的时候，我们可以通过dlv工具动态的去调试对应的**goroutine**

如下，我使用最经典的 生产者-消费者模型来使用dlv的多线程调试。

生产者：

```go
func Multi_Productor(ch chan<- int) {

	var i int = 2
	for {
		i++
		if IsPrime(i) {
			ch <- i
			time.Sleep(time.Second)
		}

	}
}
```

生产者附属生产方法:  不断的判断传入的数值是不是素数

```go
func IsPrime(number int) bool {

	for i := 2; i < number; i++ {
		if number%i == 0 {
			return false
		}
	}
	return true
}
```

#### 消费者

```go
func Multi_Receiver(ch <-chan int) {
	for value := range ch {
		fmt.Printf("%d is prime \n", value)
	}

}
```

#### 主main函数

```go
func main() {
	ch := make(chan int, 1000)

	go Multi_Productor(ch)
	go Multi_Receiver(ch)
	time.Sleep(time.Hour)

}
```

在主函数中，启用两个**goroutine**去生产和消费对象。在我们启动了以后，我们照旧可以用dlv debug 调试，同时在dlv中使用:

```bash
goroutines
```

可显示程序中运行的所有线程，当然在go里面都是轻量级的线程由go内核进行管理。

![调试查看所有的goroutine](/png10.png)

里面显示的很多的是go内核的goroutine，而跳入到某个特定的goroutine只需要使用 

```
goroutine 19.   //跳入到19号生产者线程 进行调试
```

查看调用堆栈，    **bt**    指令，会显示当前的goroutine调用的所有堆栈。

好啦，delve工具就分享到这啦，后面如果还有新的会更新在这。