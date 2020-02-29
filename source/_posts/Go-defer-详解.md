---
title: Go defer 详解
date: 2019-04-21 20:12:28
tags: [Go,服务端开发]
categories: Go栈
---

# Defer详解

![](/defer.png)

defer语句是Go中一个非常有用的特性，可以将一个方法延迟到包裹该方法的方法返回时执行，在实际应用中，defer语句可以充当其他语言中try…catch…的角色，也可以用来处理关闭文件句柄等收尾操作。

Go官方文档中对defer的执行时机做了阐述，分别是。

- 包裹defer的函数返回时
- 包裹defer的函数执行到末尾时
- 所在的goroutine发生panic时

如下，defer相当于一个临时的函数调用栈，在一个函数中，每次遇到的defer语句都会被放在一个临时的函数栈中，在函数返回之前进行调用，具体的调用时机是在**返回值真正返回之前**。

为什么这么说呢，因为Go的函数返回值返回操作并不是原子性操作，它分为两步，如下的操作：

```go
return x
```

## 返回值原理:

- 返回值=x
- RET指令

### defer操作

1. 返回值 =x
2. 检索defer函数栈，执行pop并执行
3. RET指令



接下来这个案例很能说明问题：

```go
package main

import "fmt"

func Defer() int {

	number := 10

	defer func() {
		number += 20
	}()

	return number
}

func main() {

	n := Defer()

	fmt.Print(n)

}
```

**输出的结果是： 10**

这是因为在defer之前返回值已经设置好了，所以执行的defer并不修改返回值。



再如：

```go
package main

import "fmt"

func Defer() (x int) {

	defer func() {
		x += 20
	}()

	return 5
}

func main() {

	n := Defer()

	fmt.Print(n)

}

```

**输出结果： 25**

是不是有点出乎意料，这是因为在 Defer( ) 的定义里面 x作为返回值已经指定，所以其实底层执行的是：

```go
1. x = 5
2. x+=20
3. return x
```



而：

```go
package main

import "fmt"

func Defer() (x int) {

	y:=10
	defer func() {
		y += 20
	}()

	return y
}

func main() {

	n := Defer()

	fmt.Print(n)

}
```

**输出结果：10**

是不是觉得更奇怪啦，出乎意料的结果是吧！ 当然这些都是一些障眼法啦，只要了解返回值和defer的操作一下切都会显得很自然了。

```go
1. x = y(10)
2. defer()->不修改y，只修改副本
3. 返回x(10)
```

最后：

```go
package main

import "fmt"

func Defer() (x int) {

	y:=10
	defer func(x int) {
		x+=1
	}(y)

	return y
}

func main() {

	n := Defer()

	fmt.Print(n)

}
```

**输出结果： 10**

这个就比较简单，匿名函数传入的是值的副本，所以修改不到原值。



好啦，defer到此结束～