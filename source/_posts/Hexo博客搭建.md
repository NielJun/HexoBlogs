---
title: Hexo博客搭建
date: 2018-01-06 23:39:42
tags: 
categories: Go栈
---


# Hexo背景

写blog虽然经历了N多不同时代的产品，恒久不变的始终是自己无人问津的网站。虽然没几个人看，还是隔断时间就要折腾一下。从最开始的wordpress，到tale，到现在的hexo，网站变得越来越简单，越来越轻量级，这里主要说说hexo的搭建和使用。

## Hexo介绍

[Hexo系统的主页]( https://hexo.io/zh-cn/)有非常详细的介绍，主页中有介绍内容我这里就不赘述了，这里主要说说主页中没有详细说明内容。

- hexo 可以理解为是基于node.js制作的一个博客工具，不是我们理解的一个开源的博客系统。其中的差别，有点意思。
- hexo 正常来说，不需要部署到我们的服务器上，我们的服务器上保存的，其实是基于在hexo通过markdown编写的文章，然后hexo帮我们生成静态的html页面，然后，将生成的html上传到我们的服务器。简而言之：hexo是个静态页面生成、上传的工具。

## Hexo的安装

1. 安装nodejs  

   ```bash
   sudo su  //切换到管理员
   npm install -g cnpm --registry=https://registry.npm.taobao.org. //安装cnpm 工具
   ```

   

2. npm安装hexo博客

   ```bash
   cnpm install -g hexo-cli
   ```

3. 创建一个初始化博客的文件夹. pwd 查看当前目录位置 使用

   ```bash
   mkdir hexoblogs  创建目录
   ```

   当前目录是 /Usrs/daniel下

4. 使用 cd hexoblogs 进入目录进行博客初始化 :

   ```bash
   sudo hexo init
   ```

5. 启动hexo博客 默认是4000端口 

   ```bash
   hexo s 
   ```

   至此，hexo博客可以正常启动了，可以通过 [本地回路](localhost:4000) 访问博客

   ![](/start.png)

### 博客文件夹的目录结构

- | 文件/文件夹 | 说明                                                  |
  | ----------- | ----------------------------------------------------- |
  | _config.yml | 配置文件,用于修改相关的服务器生成配置                 |
  | public      | 生成的静态文件，这个目录最终会发布到服务器            |
  | scaffolds   | 一些通用的markdown模板                                |
  | source      | 编写的markdown文件，_drafts草稿文件，_posts发布的文章 |
  | themes      | 博客的模板主题                                        |

### Hexo常用命令

- Hexo 创建新的博客

  ```bash
  hexo n "博客名字"
  ```

  

- Hexo 清除相关记录

  ```bash
  hexo clean / hexo cl /hexo c 每个都可以
  ```

  

- Hexo 重新新城相关博客文件

  ```bash
  hexo g
  ```

  

- Hexo 启动

  ```bash
  hexo s
  ```



## 博客部署

##### 由于Hexo是静态的博客系统，所以可以通过Github仓库托管并且部署，当然也可以通过国内的码云进行托管，毕竟速度甩Github几条街。当然也可以部署在自己的服务器上面。

- ### Github托管方式

  1. 在自己的仓库里面新建一个response 

     ```bash
     注意：名字是固定的 是自己的  Github名字 + .github.io
     如 ： NielJun.github.io
     ```

  2. 在hexoblogs目录下安装git插件

     ```bash
     命令为： cnpm install --save hexo-deployer-git
     ```

  3. 然后打开当前目录下面的_config.yml 

     ```bash
     vim _config.yml 
     ```

  4. 在文件底部修改![](/hexo_config.png)

     其中type为git  repo 为创建的仓库地址 branch为master

  5. 最后将本地修改好的博客部署提交到github

     ```bash
     hexo d
     ```

  Github部署就完成啦。

- ### Gittee 码云仓库部署

  待更新

- ### **个人服务器部署**  

  ​		步骤其实很简单，就是在自己的服务器上**建一个仓库**，然后通过**nginx反代**，**本地修改提交的目录**即可

1. 通过ssl到自己的后台，我的云服务器是centeros系统，安装git

   ```bash
   yum install git
   ```

   

2. 添加git用户

   ```bash
   useradd git
   ```

   

3. 重新设置一下git仓库密码，当然不设置的话密码是无效的。

   ```bash
   passwd git
   输入两次验证即可更新令牌
   ```

   

4. 切换到git用户

   ```bash
   su git
   ```

   

5. 管理hexo在云服务器上的目录地址： 

   ```bash
   cd /home/git //切换到git目录
   mkdir -p HexoBlogs 	//新建博客目录
   mkdir repos && cd repos			//新建并进入新建的仓库地址
   ```

   

6. 新建一个 bare [空的] 仓库

   ```bash
   git init --bare HexoBlogs  			 // --bare表示空的
   cd HexoBlogs/Hooks 							 // 进入hooks目录
   ```

   

7. 编辑（或者是创建) post-receive 文件

   ```bash
   vim post-receive
   ```

   ```bash
   //编辑内容如下
   #!/bin/sh
   git 		--work-tree=/home/git/HexoBlogs       --git-dir=/home/git/repos/HexoBlogs.git checkout -f
   ```

   

8. 修改刚刚新建的文件的权限

   ```bash
   chmod +x post-receive
   ```

   

9. 退出Hooks

   ```bash
   exit
   ```

10. 修改仓库权限

    ```bash
    chown -R git:git /home/git/repos/HexoBlogs.git/
    ```

11. 【**测试步骤**】服务器上面的地址新建完成，可以在本地机器上面实验是否可以拉取到Git 中间段是服务器ip

    ```bash
    git clone git@175.24.15.152:/home/git/repos/HexoBlogs.git test/blogs
    ```

    

12. **这里插一句，游戏我们的本机和服务器建立的是访客关系每次登陆都需要密码，我们可以建立一个私有关系，让登陆无密码。**[**此部分可跳过 第12步**]

    1. 准备ssh-copy-id

       ```bash
       brew install ssh-copy-id
       ```

       

    2. 生成私钥公钥

       ```bash
       ssh-keygen -t rsa -b 1024
       ```

       

    3. 将公钥上传致服务器(~/.ssh/authorized_keys)

       ```bash
       ssh-copy-id -i 公钥 root@175.24.15.152
       ```

       

    4. 在本地~/.ssh/config文件中添加(如果没有则新建)

       ```bash
       Host git.oschina.net
       
       IdentityFile ~/.ssh/weiwei.sun
       
        
       
       Host 192.168.50.100
       
       IdentityFile ~/.ssh/weiwei.sun
       ```

       

    **注:** 不过在mac下这里需要 **brew** 附带其安装和卸载吧

    - **安装brew**

    ```bash
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```

    - **卸载brew**

      ```bash
      /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
      ```

      

13. 在本地拉取完成以后，直接修改本地hexo的配置

    ```bash
    # Deployment
    ## Docs: https://hexo.io/docs/deployment.html
    deploy:
      type: git
      #repo: https://github.com/NielJun/NielJun.github.io.git     //之前的在github上托管
      repo: git@175.24.15.152:/home/git/repos/HexoBlogs.git				//放在自己的服务器上
      branch: master
    ```

14. 最后一步，nginx反代，关于nginx可以参考我的nignx相关博客。

    ```bash
    修改的 nginx.config条目
    
    	a.  顶端取消注释修改为  user root  开启权限
    	b.	在server区  server_name 修改为ip
    	c.	server{location{}} 中间
    	
             location / {
                      root   /home/git/HexoBlogs/;
                      #proxy_pass http://175.24.15.152:5897/blogs/NielJun;
                      index  index.html index.htm;
                  }
    
    
    ```

    再重新热更配置:

    ```bash
    nginx -s reload
    ```

    本地照常修改和生成博客，只是提交以后就是在服务器上自己的git仓库了。

    至此，可以直接用ip/域名访问到[博客](http://www.rubyboy.cn)了，皆大欢喜。

## Hexo换主题

![](/hexothemes.jpg)

hexo更换主题很简单，注需简单的几步：

1. 先找到一个主题的git仓库 如 [hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia.git)

2. 把其拉到themes目录下面 

   ```bash
   git clone https://github.com/litten/hexo-theme-yilia.git  themes/yilia
   ```

   

3. 修改_config.yml文件 把其中的them条目改成yilia 保存退出 这个配置文件是博客系统根目录下面的_config.yml

4. 本地预览没问题以后再hexo d 推送到远端

   ```bash
   hexo clean
   hexo g
   hexo s 本地查看
   hexo d 推送到远端
   ```

   

   