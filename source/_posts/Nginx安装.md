---
title: Nginx服务器之一 安装
tags: [Go,服务端开发,Ngix]
categories: Go栈
---
​      				![](/nginx.jpg)

> Nginx是一款轻量级的Web 服务器反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强，是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。

## nginx安装

- 推荐使用官网的压缩包解压再上传到linux进行安装

###  进入 /usr/local/sbin/目录

``` bash
$ cd  /usr/local/sbin/
```

### 安装相关依赖

``` bash
$ yum -y install make zlib-devel gcc-c++ libtool  openssl openssl-devel
```

### 官网下载相关的linux安装包

[Nginx官网](http://nginx.org/en/download.html)

More info: [Generating](https://hexo.io/docs/generating.html)

### 在linux上进入

``` bash
$ cd /usr/local/src/
```

### 然后把压缩包解压，解压命令如下

```bash
 tar -xvf nginx-1.17.7.tar
```

### 执行./nginx 启动nginx服务 启动完成以后 查看linux进程查看相关nginx启动情况

```bash
 ps -ef | grep —nginx
```

![Nginx后台进程](/bash.png)

### 由于nginx默认的是80端口 所以我们可以直接访问ip地址看nginx是否启动成功浏览器打开ip: 175.24.15.152 出现如下 则表示启动成功

![Nginx初次启动成功](/start.png)

### 由于很多的linux系统的端口是被防火墙关闭的 所以我们得带开或者查看当前端口的情况

#### 显示所有的端口情况

```bash
firewall-cmd - -list -all 
```

####  开启防火墙的8001端口

```bash
sudo firewall-cmd - -add-port=8001/tcp —permanent
```

#### 重启防火墙修改才生效

```bash
firewall-cmd-reload 
```

### Nginx 操作的常用命令

#### 注意:使用命令先要进入 /usr/local/nginx/sbin/ 目录才可使用

- 查看版本号

  ```bash
   ./nginx -v
  ```

  

- 关闭nginx

  ```bash
  ./nginx -s  stop 
  ```

  

- 启动 nginx

  ```bash
  ./nginx		
  ```

  

- 修改了配置文件[ conf/nginx.conf ]后,无需重启服务器及生效的命令 [ 重加载命令 ]

  ```
  ./nginx -s reload
  ```

  



