---

title: Go单元测试和压力测试
date: 2020-01-20 09:27:03
tags: [Go,服务端开发,软件测试]
categories: Go栈	
---

![](/start.png)

# Go语言中的自动化测试框架

​		testing包提供了自动化测试相关的框架，只需要在文件中引用测试包即可。

```go
import(
testing
)
```

支持单元测试和压力测试。以

```go
go test
```

驱动测试案例的执行。

## Go测试框架规范

由于testing是一个自动化测试框架，所以我们在编写测试用例的时候需要遵循相对应的规范。

- 用来测试的代码必须以   **_test.go**   结尾
- 单元测试的函数名必须以.   **Test**   开头，并且只有一个参数，类型是 ***Testing.T**
- 基准测试或者压力测试必须以.   **Benchmark**   开头，并且只有一个参数，类型是 ***Testing.B**

## 基础的单元测试用例的编写

新建两个go代码文件

```go
Add.go						 // 代码文件
Add_test.go.       // 单元测试文件
```

其中：

**Add.go** 里面包含一个基础的函数：

```go
func Add(a int,b int) int  {
	return a+b
}

```

Add_test.go里面包含一个基础的单元测试用例：

```go
func TestAdd(t *testing.T) {
	var a = 20
	var b = 30

	c:= Add(a,b)

	if c!=30{
		t.Fatal("invaild : a+b")  // 错误日志 执行此条panic 程序终止
	}
  
  t.Logf("a=%d b=%d,sum=%d",a,b,c)  // 调试日志 用于输出
}

```

别忘记引用测试包

```go
import "testing"
```

此时执行 

```go
go test 
```

![](/png1.png)

单元测试走了panic分支，因为这也是测试的预期。接着修改一下测试用例：

```go
func TestAdd(t *testing.T) {
	var a = 20
	var b = 30

	c:= Add(a,b)

	if c!=50{
		t.Fatal("invaild : a+b")  // 错误日志 执行此条panic 程序终止
	}
  
  t.Logf("a=%d b=%d,sum=%d",a,b,c)  // 调试日志 用于输出
}
```

![](/png2.png)

此时走的是正常的分支，不会出现panic，但是并不会出现任何的Log输出信息，因为默认情况下，t.Logf并不会直接把日志显示在终端，我们应该调用

```go
go test -v
```

OK,结果如下：

![](/png3.png)

go test 会执行**所有的以_test结尾的测试测试文件**中**所有的以TestFunc 为标准的所有测试用例**。

### 压力测试 [ Benchmark ]

我们自己写的一些通用库通常需要做相应的压力测试，testing包里面包含很好的压力测试条件和环境，使用起来也非常的简单。只需要在测试文件中相对应的测试用例名字以**Benchmake**开头，如：

```go
func BenchmarkAdd(b *testing.B) {
  // N为测试框架自带的测试压力基准值
	for i := 0; i < b.N; i++ {
		var a = 10
		var b = 20
		Add(a,b)
	}
}
```

如上代码，我们对Add函数执行压力测试，其中的N是框架传入的测试压力基准值，我么可以通过它做相应的轮询压力测试。

测试的参数和单元测试不同的是，压力测试使用的是：

```go
go test -bench .
```

执行结果：

![](/png4.png)

### 测试相关的参数

- go test -bench . 									 // 执行所有的压力测试案例
- go test -bench BenchmarkAdd  			//执行相对应的压力测试案例
- go test -run TestAdd  						     //执行特定的测试案例
- go test -v												//查看对应的测试详情