---
title: Mac设置文件权限问题
date: 2020-02-08 10:35:21
tags: [Mac,文件权限]
categories: Tips
---

### Mac设置文件权限的问题

​		在使用mac时，经常我们遇到相关文件不能使用的情况，其实大多数情况都是，文件权限问题。

　　文件或目录的访问权限分为只读，只写和可执行三种。以文件为例，只读权限表示只允许读其内容，而禁止对其做任何的更改操作。可执行权限表示允许将该文件 作为一个程序执行。文件被创建时，文件所有者自动拥有对该文件的读、写和可执行权限，以便于对文件的阅读和修改。用户也可根据需要把访问权限设置为需要的 任何组合。

​		比如使用Hexo生成博客文件的时候，在Mac上默认是只读的文件。

![](/1.png)

我们可以右键在简介里面修改，但是很繁琐。这里介绍一个权限命令可以一键修改，方便有用到的人使用。

mac设置文件夹权限

```
chmod -R 777 testfile
```

　　设置文件权限

```
chmod 777 txt.txt
```

　　

　　1，禁止.DS_store生成：打开  “终端” ，复制黏贴下面的命令，回车执行，重启Mac即可生效。　　

　　禁止.DS_store 会导致文件信息记录丢失。

```
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
```

　　2，恢复.DS_store生成：

```
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```

 

　　删除文件 rm file  只是删除文件夹里面没有嵌套文件夹 rm -r folder   反之则 rm -rf folder　　