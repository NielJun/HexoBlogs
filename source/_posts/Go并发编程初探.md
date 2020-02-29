---
title: Go并发编程初探
date: 2020-01-20 19:17:11
tags: [Go,服务端开发,并发编程]
categories: Go栈 
---

### 并发与并行

并发：同一时段内执行多个操作,如两排公用一台咖啡机，在时间粒度为1h时的情况下，我们可以把它看作是一个人眼可见的并发模型，当然这只是个比喻。

![](/png1.png)

并行：同一时刻执行多个操作，这里就涉及到两个咖啡机同时处理操作。

![](/png2.png)

明白了并发和并行的话，接下来就好说。

**多线程技术**

- 线程是有操作系统管理的一组程序的执行流，是操作系统能操作的最小单位。简而言之，线程是一个程序运行时[进程]产生的一个小分支，一个进程中可以并发多个线程，每条线程并行执行不同的任务。线程是独立调度和分派的基本单位。线程可以为操作系统内核调度的**内核线程**和由用户进程自行调度的**用户线程**
- 在很多的编程语言里面，多线程技术的实现是一个抢占式的模型，每一个属于应用层面的线程都是附属于该应用的一个运行分支，通过cpu来进行调度，但是在操作系统层面，不断的从内核态到应用态的切换会使得CPU的处理消耗得更大，简而言之，直接通过操作系统调度不断切换的线程，因为涉及到当前线程从被挂起到得到时间片恢复会浪费很大的资源。所以在java里面对于多线程的使用都会使用线程池来进行优化，但是这种优化也是有瓶颈的，毕竟只是在运用层面。
- Go语言的**goroutine技术**是类似于**unity的协程**技术，但是与之不相同的事，他是在语言层面在Go运行时的内核里面做自己的调度，所以当CPU给到进程时间片的时候，内核会直接对进程树下面的所有的微线程的[goroutine]进行调度，减少了操作系统层面的上下文切换，非常的轻量级。

### Goroutine原理

简而言之，一个操作系统会对应用户态多个goroutine，而且会使用多个操作系统进线程，而且操作系统线程对于goroutine是 **M：N**的关系。

做一个模型抽象：

- 造作系统线程设为：  M
- 用户态线程 [ goroutine ] 设为:    G
- 上下文对象设为：    P

![](/png6.png)

其中G会被M去执行，P相当于执行的中间件，也就是M执行G任务的时候P是执行环境，只是类似于操作系统内核态的上下文，只是足够的轻量级。

### Goroutine调度

![](/png7.png)

当前M执行G任务时，其他的G会进入等待状态，当当前的G被调度完成或者时间片轮询超时，此时的G会被放入队尾，队头任务被调度，依次轮询执行，当然这里面还有一些其他的调度算法。

### 当用户态线程阻塞时如何调度？

由于我们的物理线程会对应执行多个用户态线程(Goroutine)，当我们的某个goroutine进行一个耗时操作的时候，比如：

![](/png8.png)

当当前Mo调度Go进行耗时的文件操作时，如果调度依旧进行，则会出现接下来的所有的被Mo调度的goroutine都会‘卡死’，所以呢，Go在运行时内核为这样的耗时操作新开一个调度，然后单一的去执行耗时操作，其余的依旧轮询使用时间片调度。

### Goroutine池

当我们在进行某些并发操作计算的时候，每次遇到都开启新的goroutine去执行，难免会出现goroutine的暴增，使用不当甚至出现泄漏的情况。通常使用goroutine池来进行任务的调配和线程间通信。

比如一个非常简单的例子，**随机不断生成随机数，然后启动worker池来进行计算每个数的各个位上的数的和并且输出。**

> number: 1234 , result: 1+2+3+4 = 10

开两个抽象类进行封装：

```go
type Job struct {
	JobId  int
	Number int
}

type Result struct {
	Job *Job
	sum int
}
```

一个是表示一个计算任务，一个存储计算结果。

每一个worker都回去做计算任务，并且把计算的结果放入结果队列。

```go
// 求出每个位上的值的和 如1234 = 1+2+3+4 = 10
func (job *Job) Work(result chan *Result) {

	//计算
	sum := 0
	number := job.Number

	for number != 0 {
		temp := number % 10
		sum += temp
		number /= 10
	}

	//把计算的结果变成对象
	r := &Result{
		Job: job,
		sum: sum,
	}

	//写入管道
	result <- r

}

```

而每一个Worker负责从任务队列里面取出任务，然后进行计算。

```go
func Worker(jobChan chan *Job, resultChan chan *Result) {

	for job := range jobChan {
		job.Work(resultChan)
	}
}

```

再封装一个启动器,表示启动多大的worker池。

```go
//启动worker池
func StartWorkerPool(poolSize int, jobChan chan *Job, resultChan chan *Result) {
	for i := 0; i < poolSize; i++ {
		go Worker(jobChan, resultChan)
	}
}
```

用于产生随机数的接口和输出结果的接口是用来做显示的，这也列出来。

```go
func ProductJobs(jobChan chan *Job) {
	jobId := 1
	for {
		jobId++

		number := rand.Int()

		job := &Job{
			JobId:  jobId,
			Number: number,
		}

		jobChan <- job

	}

}

func PrintWorker(resultChan chan *Result) {
	for result := range resultChan {
		fmt.Printf("JobId : %d number: %d  result : %d \n", result.Job.JobId, result.Job.Number, result.sum)
	}
}

```

最后只需在main里面进行调用即可。

```go
func main() {

	jobChan := make(chan *Job, 1024)
	resultChan := make(chan *Result, 1024)

	//启动worker池
	StartWorkerPool(512, jobChan, resultChan)

	go PrintWorker(resultChan)

	go ProductJobs(jobChan)

	select {}
}
```

结果；

![](/png9.png)

这是个非常简单的例子，但是仔细看会觉得很优雅，这个就是使用goroutine实现像工厂流水线一般的运作方式。

### 线程安全

首先，Go语言在设计理念上已经做到了足够简单，我们无须像C/C++，Java等语言在设计多线程的时候考虑各种线程之间的问题以及多线程管理带来的麻烦。但是，只要有并发的概念，那就避免不了资源冲突的问题。我以前在设计游戏的时候，因为有些游戏的逻辑比较复杂，用到的协程非常多，导致很多的资源在我某个协程取它的时候已经实现被其他协程修改了。

比如说： 我有两个协程，一个是用来计算修改子弹的伤害，一个是用来计算子弹销毁。如果两个协程不自己进行处理的话，很多时候子弹就会突然消失。当然游戏设计并不是说的这么简单，中间还有很多的逻辑。

所以我们需要在两个同时想持有某项资源的线程做一个限定，不能让操作同时发生，出现意料之外的结果。而这样的被多个线程所持有的资源没称作临界资源或临界区。

```go
x = x+10
```

这是一个看起来再简单不过的语句，但是要知道在并发的世界里面，什么都没这么简单，我们分析一下，在程序底层实际上是做了三步**原子操作**：

1. 先从内存中取出x的值
2. CPU进行计算 x+10
3. 把x+10存储到内存中

为什么这么分析，因为中间的每一步都有可能被别人取先。 所以我们需要加锁而避免出现临界资源在多个线程同时操作下出现问题。

![](/lock.jpg)

### 互斥锁

当对某个资源进行非原子操作的时候，我们对此操作加锁，使得其他的线程对其操作的时候只能等待其锁被释放后才能进入临界区，而多个线程等待的时候，其唤醒策略是随机的，或者看线程的优先级而定。

```go
var mutex sync.Mutex   // 在全局区定义互斥锁

mutex.Lock()
i++
mutex.Unlock()

```

使用很简单，只需在临界操作前加锁，操作后解除锁即可。

### 读写锁

在我们对临界区资源进行读操作的时候其实是不用加锁的的，加锁反而会降低性能。但是当我们需要读取值的时候却中途被雪入了其他不正确的值，就会出现错误。所以**读写锁**应运而生。

**读锁：**

- 当一个goroutine获得读锁以后，**其他的goroutine获取写锁都会等待。**
- 当一个goroutine获得读锁以后，**其他的goroutine获得读锁时，都会获得锁。**

**写锁：**

- 当一个goroutine获得写锁以后，**其他的goroutine获取写锁和读锁都会等待,写锁是互斥的**

```go
var rwLock sync.RWMutex

//读锁
rwLock.RLock()
fmt.Println("a = ",a)
rwlock.RUnlock()


//写锁
rwLock.Lock()
a++
rwlock.Unlock()
```

在读多写少的情况下，读写锁的性能比互斥锁高很多。

### 原子操作

为什么要介绍原子操作呢？这个原子操作和上面讲的单步的原子操作是不一样的两件事，这个原子操作是Go语言里面的一种单步操作的函数。由于**加锁代价比较高，比较耗时，需要上下文切换**。所以在Go语言里面，针对基本数据类型可以使用原子操作保证线程安全。原子操作可以在用户态就可以完成，所以性能比互斥锁高。

在使用互斥锁的时候：

```go
var mutex sync.Mutex   // 在全局区定义互斥锁

mutex.Lock()
i++
mutex.Unlock()
```

原子操作：

```go
atomic.AddInt32(&x,1)  // 相当于x = x + 1
```

我对10000个goroutine进行原子操作和互斥锁操作，原子操作的性能是互斥锁的5倍左右，锁着goroutine的数量级越大，性能差异也就越大。

相对应的其他原子操作：

```go
a := atomic.LoadInt32(&x)   						//  加载操作.  var a int = x 
atomic.StoreInt32(&x,10)   						  //  写入. x = 10
b := atomic.SwapInt32(&x,10)					  //  交换操作
```

