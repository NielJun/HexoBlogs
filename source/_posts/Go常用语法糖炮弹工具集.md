---
title: Go常用语法糖炮弹工具集
date: 2020-01-10 12:52:21
tags: [Go,服务端开发]
categories: Go栈
---

![](/p1.jpg)

### 关于Go

Go语言虽然说是C族语言，也被调侃为21世纪网络C语言，但是Go不得不说它是一种简单到特立独行的语言。

```go
var age int
name := "小鬼 Boy"
```

万种风流，都放在类型后面的，它放在前面..

```go
type Animal interface {
	Name()
	Say()
	Eat()
}
```

```go
type Dog struct {

}

func (dog * Dog)Name()  {
	fmt.Println("我叫旺财")
}

func (dog * Dog)Say()  {
	fmt.Println("旺旺")
}

func (dog * Dog)Eat()  {
	fmt.Println("哇哦")
}
```

没有Class概念，却有着非同寻常的interface，可以说Go是组合优于继承的集大成者。

更像一个100斤小孩子的是， 时间格式化的方式居然固定在Go语言的生日！！wtf？ 写错一个数字居然会有意想不到的结果...

```go
nowStr := now.Format("2006-01-02 15:04:05.999")  //固定数字 格式可以随便搭配
```

另外，在编写日志库系统的时候，难免会用到取得当前crash的位置：

```go
func GetLineInfo(fileName string,funcName string,lineNumber int){
	pc,file,line,ok := runtime.Caller(0)
	if ok{
		fileName = file
    //pc转函数
		funcName = runtime.FuncForPC(pc).Name()
		lineNumber = line
	}
	return
}
```

pc是当前运行到的指令计数器
runtime.Caller(0)后面的**参数**是表示**函数调用栈**里面的函数的所在行，所以当参数为 0 时代表的是当前GetLineInfo（）的函数所在行，当参数为 1 时表示GetLineInfo的被调函数头所在行，以此类推。

```go
10 	fuc DropParent(){
11    	fileName, funcName, lineNumber := GetLineInfo()
12 	}

14 fun DropGrandParent(){
15  DropParent()
16 }
```

当：

```
runtime.Caller(0)时
```

返回的是函数栈栈底函数的所在地址，即 func GetLineInfo(fileName string,funcName string,lineNumber int)所在行，

当

```go
runtime.Caller(1)时
```

返回的是函数栈栈底第二个函数的所在地址，即 fuc DropParent()所在行，如此往复。

先写到这，以后再写更多的go的一些常用到但是每次还要去查阅的相关的东西。