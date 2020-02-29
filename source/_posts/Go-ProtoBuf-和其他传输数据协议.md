---
title: Go ProtoBuf 和其他传输数据协议
date: 2020-01-25 21:44:42
tags: [Go,网络协议]
categories: Go栈
---

![](/start.png)

## 常用的数据传输协议

为什么要有数据传输协议呢，其实这个问题非常简单，像我们在做游戏开发的时候，比如游戏引擎层面的开发语言是用的Lua、C++或者C#等，但是往往在服务器上用的并不是对等的语言，可能会在小项目的时候用Java写服务器，随着规模的增大，我们可能会考虑升级到像C++或者Node等。这样者皆适用Socket在Tcp/Udp等协议的支持下，虽然说客户端和服务器连接不成问题，但是，客户端和服务器两种不同的语言的数据类型很不一样，直接用流式传输字节流已经显得力不从心了。

假若我们采用一种统一的中间格式做数据中介，只是在传输钱序列化，在接受后反序列化成特定语言的类型就不会出现因为类型不匹配而出现的困扰了。

### Json

> JSON(JavaScript Object Notation,JavaScript对象表示法)是一种轻量级的、键值对的数据交换格式。结构由大括号’{}’，中括号’[]’，逗号’，’，冒号’;’，双引号’""'组成，包含的数据类型有Object，Number,Boolean,String,Array, NULL等。

![](/png4.png)

json图示意：

![](png1.png)



#### json序列化

> 将Go语言原数据转换成JSON格式字符串

```go
//传map,结构体,slice...,返回结果byte切片和error是否错误
func Marshal(v interface{}) ([]byte, error)

```

#### 结构体转json

```go
type Person struct{
   Name string   //姓名
   Age int       //年龄
   Sex rune      //性别
   Hobby []string  //爱好
   Money float64   //钱
}
  
person:=Person{Name:"张三",Age:18,Sex:'男',Hobby:[]string{"听音乐","看书","打篮球"},Money:18.62}
if bytes,err:=json.Marshal(person);err!=nil{
  fmt.Println("编码错误",err)
}else{
//{"Name":"张三","Age":18,"Sex":30007,"Hobby":["听音乐","看书","打篮球"],"Money":18.62}
  fmt.Println("编码成功:",string(bytes))
}
```

#### Map转Json

```go
  p:=make([]map[string]interface{},0)
  p1:=map[string]interface{}{"name":"张三","age":18,"sex":'男',"hobby":[]string{"听音乐","看书","打篮球"},"money":18.62}
  p2:=map[string]interface{}{"name":"李四","age":19,"sex":'女',"hobby":[]string{"听音乐","看电影","打足球"},"money":1.62}
  p=append(p,p1,p2)

  if bytes,err:=json.Marshal(p);err!=nil{
    fmt.Println("编码错误",err)
  }else{
    fmt.Println(string(bytes))
  }
```

### Json反序列化

> 将JSON格式字符串转换成Go语言原数据

```go
//传入JSON字符串的byte字节和Go接收数据的类型指针，返回err错误，是否返回成功
func Unmarshal(data []byte, v interface{}) error

```

#### Json转map

```go
str:=`{"Name":"张三","Age":18,"Sex":30007,"Hobby":["听音乐","看书","打篮球"],"Money":18.62}`

p:=make(map[string]interface{}, 0)
if err:=json.Unmarshal([]byte(str),&p);err!=nil{
  fmt.Println("解码失败",err)
}else{
  fmt.Println("解析成功",p)
}
```

#### Json转结构体

```go
str:=`{"Name":"张三","Age":18,"Sex":30007,"Hobby":["听音乐","看书","打篮球"],"Money":18.62}`
var p Person

if err:=json.Unmarshal([]byte(str),&p);err!=nil{
  fmt.Println("解码失败",err)
}else{
  fmt.Println("解析成功",p)
}
```

#### Json转切片

```go
str:=`[{"Hobby":["听音乐","看书","打篮球"]}]`
p:=make([]map[string]interface{}, 0)

if err:=json.Unmarshal([]byte(str),&p);err!=nil{
  fmt.Println("解码失败",err)
}else{
  fmt.Println("解析成功",p)
}
```

Json的使用非常简便，使用的也是文本协议，人类直接可以读懂修改的文本段。

### XML

xml主要用于配置文件的编写，由于xml是第一代文本协议，解决了一些跨语言的难题，但是由于xml占用的空间比较大，资源利用率很低，所以只用在配置文件这种文件树相对简单，文件类型单一的使用上。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persons>
    <person name="polaris" age="28">
        <career>无业游民</career>
        <interests>
            <interest>做游戏</interest>
            <interest>打篮球</interest>
        </interests>
    </person>
    <person name="studygolang" age="27">
        <career>工程师</career>
        <interests>
            <interest>编程</interest>
            <interest>下棋</interest>
        </interests>
    </person>
</persons>
```

如上的xml文件如何解析呢？

首先对应的结构体定义如下：

```go
type Result struct {
    XMLName xml.Name `xml:"persons"`//标签上的标签名
    Persons []Person `xml:"person"`
}
type Person struct {
    Name string `xml:"name,attr"`//persion标签属性名为name的属性值
    Age int `xml:"age,attr"`
    Career string `xml:"career"`//persion中标签名为career的值
    Interests []string `xml:"interests>interest"`//标签interests下的interest数组
    //不写>当子标签为一个的时候回把它当做对象解析
}
```

#### XML字符串读取

##### 方法一：转对象



```xml
input := `<Person><FirstName>Xu</FirstName><LastName>Xinhua</LastName></Person>`
err := xml.Unmarshal([]byte(input), &v)//将文件转化成对象
```

##### 方法二：遍历



```go
package main

import (
    "encoding/xml"
    "strings"
    "fmt"
)

func main() {
    var t xml.Token
    var err error

    input := `<Person><FirstName>Xu</FirstName><LastName>Xinhua</LastName></Person>`
    inputReader := strings.NewReader(input)

    // 从文件读取，如可以如下：
    // content, err := ioutil.ReadFile("studygolang.xml")
    // decoder := xml.NewDecoder(bytes.NewBuffer(content))

    decoder := xml.NewDecoder(inputReader)
    for t, err = decoder.Token(); err == nil; t, err = decoder.Token() {
        switch token := t.(type) {
        // 处理元素开始（标签）
        case xml.StartElement:
            name := token.Name.Local
            fmt.Printf("Token name: %s\n", name)
            for _, attr := range token.Attr {
                attrName := attr.Name.Local
                attrValue := attr.Value
                fmt.Printf("An attribute is: %s %s\n", attrName, attrValue)
            }
        // 处理元素结束（标签）
        case xml.EndElement:
            fmt.Printf("Token of '%s' end\n", token.Name.Local)
        // 处理字符数据（这里就是元素的文本）
        case xml.CharData:
            content := string([]byte(token))
            fmt.Printf("This is the content: %v\n", content)
        default:
            // ...
        }
    }
}
```

#### XML文件读取



```go
//从文件读取，如可以如下：
content, err := ioutil.ReadFile("studygolang.xml")
decoder := xml.NewDecoder(bytes.NewBuffer(content))
xml.Unmarshal(content, &result)//将文件转化成对象
```

### 生成xml



#### 对象转换为xml



```go
package main

import (
    "encoding/xml"
    "fmt"
    // "os"
)

type Servers struct {
    XMLName xml.Name `xml:"servers"`
    Version string   `xml:"version,attr"`
    Svs     []server `xml:"server"`
}

type server struct {
    ServerName string `xml:"serverName"`
    ServerIP   string `xml:"serverIP"`
}

func main() {
    v := &Servers{Version: "1"}
    v.Svs = append(v.Svs, server{"Shanghai_VPN", "127.0.0.1"})
    v.Svs = append(v.Svs, server{"Beijing_VPN", "127.0.0.2"})
    output, err := xml.MarshalIndent(v, "  ", "    ")
    if err != nil {
        fmt.Printf("error: %v\n", err)
    }
    // os.Stdout.Write([]byte(xml.Header))

    // os.Stdout.Write(output)
    //将字节流转换成string输出
    fmt.Println("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n"+string(output))
}
```

### map与XML相互转换（只能解决简单的的xml）



```go
package main

import (
    "encoding/xml"
    "fmt"
    "io"
)

type Map map[string]string

type xmlMapEntry struct {
    XMLName xml.Name
    Value   string `xml:",chardata"`
}

// MarshalXML marshals the map to XML, with each key in the map being a
// tag and it's corresponding value being it's contents.
func (m Map) MarshalXML(e *xml.Encoder, start xml.StartElement) error {
    if len(m) == 0 {
        return nil
    }

    err := e.EncodeToken(start)
    if err != nil {
        return err
    }

    for k, v := range m {
        e.Encode(xmlMapEntry{XMLName: xml.Name{Local: k}, Value: v})
    }

    return e.EncodeToken(start.End())
}

// UnmarshalXML unmarshals the XML into a map of string to strings,
// creating a key in the map for each tag and setting it's value to the
// tags contents.
//
// The fact this function is on the pointer of Map is important, so that
// if m is nil it can be initialized, which is often the case if m is
// nested in another xml structurel. This is also why the first thing done
// on the first line is initialize it.
func (m *Map) UnmarshalXML(d *xml.Decoder, start xml.StartElement) error {
    *m = Map{}
    for {
        var e xmlMapEntry

        err := d.Decode(&e)
        if err == io.EOF {
            break
        } else if err != nil {
            return err
        }

        (*m)[e.XMLName.Local] = e.Value
    }
    return nil
}

func main() {
    // The Map
    m := map[string]string{
        "key_1": "Value One",
        "key_2": "Value Two",
    }
    fmt.Println(m)

    // Encode to XML
    x, _ := xml.MarshalIndent(Map(m), "", "  ")
    fmt.Println(string(x))

    // Decode back from XML
    var rm map[string]string
    xml.Unmarshal(x, (*Map)(&rm))
    fmt.Println(rm)
}
```

### msgpack

msgpack是针对json做的优化传输，相当于是一个二进制的json。上面两种Json和Xml格式，都是文本格式的数据，好处在于能够方便的阅读。但是问题在于占用空间比较大。所以又出现了MsgPack这种格式，它是在json基础上转换为二进制进行传输的。对应关系像下面这个图：

![](/png2.png)

MsgPack并没有官方的包，我们需要使用一个第三方的包，[项目地址](https://github.com/vmihailenco/msgpack )

使用:以下代码进行拉取相关依赖包。

```bash
go get -u github.com/vmihailenco/msgpack 
```

实现比较简单，将 json.Marshal 和 json.Unmarshal 中的【 json】替换为【 maspack】即可。

### Proto Buff

![](/jpg3.jpg)

protobuf是Google公司开发出的一种数据格式。[官方文档地址](https://developers.google.cn/protocol-buffers/ )。

简单讲它使用了**IDL语言**作为中间语言来串联不同的编程语言。不同的语言可以根据生成的IDL中间语言，生成自己的语言。

这样做有什么好处？ 举个例子：当我们在协作开发的时候，A部门使用的是Go语言、B部分使用的是Java语言，C部门使用的是C#语言，当他们之间进行数据交换的时候，都要各自维护自己的结构体，才能进行数据的

序列化和反序列化，**使用protobuf的好处就是只需要一个IDL描述**，然后生成不同的语言的结构，这样维护一份就可以了。

同时 prototbuf的性能也很好，这也是它的一个优势。IDL语言使用的变长编码（根据整数的范围 0-255 那么这个数字就占用1个字节 ，如果使用定长编码的话 一个整数可能就是 4个字节）所以它的空间利用率是很好的。

那开发流程是怎样的？

A. IDL编写

B. 生成只定语言的代码

C. 序列化和反序列化

 

如何在Go中应用prototbuf

A.安装protoc编译器，解压后拷贝到GOPATH/bin目录下, [下载地址](https://github.com/google/protobuf/releases) 这个是一个可执行程序。

B.安装生成Go语言的插件

执行命令：

```bash
go get -u github.com/golang/protobuf/protoc-gen-go
```

创建简单的proto文件:

```protobuf
//指定版本
//注意proto3与proto2的写法有些不同
syntax = "proto3";

//包名，通过protoc生成时go文件时
package school;

//性别
//枚举类型第一个字段必须为0
enum Sex {
    male = 0;
    female = 1;
    other =2;
}

//学生
message Student {
    Sex sex = 1;
    string Name = 2;
    int32 Age =3;
}

//班级
message Class{
    repeated Student Students =1;
    string Name = 2; 
}
```

 message 就可以理解成类， repeated可以理解成数组。

D.利用之前下载好的protoc.exe 生成一个Go的代码。 第一个【.】代表当前输出的目录，后面*.proto则是 proto文件的路径

```bash
protoc --go_out=.  *.proto
```

protoc --go_out=.\school\ .\school.proto

执行之后会生成如下的文件，这个go文件就可以直接使用了。

 ![](/png5.png)

E. 使用生成的Go文件

①使用 proto.Marshal() 执行序列化

```go
func writeProto(filename string) (err error) {
    //创建学生信息
    var students []*school.Student
    for i := 0; i < 30; i++ {

        var sex = (school.Sex)(i % 3)
        student := &school.Student{
            Name: fmt.Sprintf("Student_%d", i),
            Age:  15,
            Sex:  sex,
        }

        students = append(students, student)
    }

    //创建班级信息
    var myClass school.Class
    myClass.Name = "我的班级"
    myClass.Students = students

    data, err := proto.Marshal(&myClass)
    if err != nil {
        fmt.Printf("marshal proto buf failed, err:%v\n", err)
        return
    }

    err = ioutil.WriteFile(filename, data, 0755)
    if err != nil {
        fmt.Printf("write file failed, err:%v\n", err)
        return
    }
    return
}
```

②使用proto.Unmarshal(data, &mySchool)执行反序列化

```go
func readProto(filename string) (err error) {
    var mySchool school.Class
    data, err := ioutil.ReadFile(filename)
    if err != nil {
        return
    }
    err = proto.Unmarshal(data, &mySchool)
    if err != nil {
        return
    }

    fmt.Printf("proto:%v\n", mySchool)
    return
}
```

如果在使用protobuf生成的Go文件，出现了如下的异常：

**undefined: proto.ProtoPackageIsVersion3**

这个时候可能是由于**上面两步下载的protoc可执行文件 和 protobuf 的版本不一致**导致的。

1. 可以**清空**下gopath下的 github.com\golang\protobuf 然后重新下载，并在github.com\golang\protobuf\protoc-gen-go 执行 **go install** 命令。

2. 检查一下是不是使用了 **godep** 等包管理工具，里面引用的版本和protoc可执行文件不一致造成的