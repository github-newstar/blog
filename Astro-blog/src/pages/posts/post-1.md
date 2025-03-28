---
layout: ../../layouts/MarkdownPostLayout.astro
title: "容器化开发环境搭建"
pubDate: 2025-03-28
description: "这是一个一个一个容器化开发环境搭建教程！"
author: "Momo"
image:
  url: "/assets/postImg/example_2.jpg"
  alt: "img1"
tags: ["astro", "blogging", "learning in public"]
---

# 介绍

    个人的开发环境是基于远程docker服务器（其实就是内网的一个windows docker desktop)和VsCode。做到了双重前后端分离：
     - 开发前端（VsCode）和开发环境的分离（Docker Server里的linux）
     - VsCode的整体框架和编辑器的分离（neovim)
     好处如下：
      - 一次配置，到处运行
    	  - 好用的插件，好看的主题集成在VsCode里面
    	  - docker里的开发环境可以简单的保存、迁移
    	  - VsCode就算G了，也不影响环境
      - vim的兼容
    	  - 使用vscode-neovim插件让neovim做VsCode的文本编辑器
    	  - vscode好用的插件和neovim好用的插件，我全都要
    	  - 更适合vimer宝宝的体质的全键盘输入支持
      - 解决了苹果内存太贵的问题（only apple can do)
    	  - 本地开发机仅充当前端，只需要运行VsCode和Docker客户端
    	  - windows组装机的内存便宜大碗
    - 坏处：
    	- 需要简单的配置
    	- 连不上Docker Server 的话，作为开发者的生涯大概就要结束了罢（悲）

让我们开始吧

# Docker

## 什么是 Docker？

    简单来说，更轻量的虚拟机，通过复用宿主机上的一些依赖，可以使用很少量的资源就能开启一个新的linux环境。

## 好处都有啥？

    - 快速拉镜像起服务
    	场景：起mysql
    	传统方案：
    		义！雾！什么年代还在手动下载安装改配置？
    	 Docker：
    		 `docker run --name mq -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql`
    		 没错，你已经配置好一个密码是123456的mySql容器了。怎么样，docker,很神奇吧？
    - 易保存、迁移
    	- 可以打包成.tar文件到处传播
    	- 可以打上标签上传到dockerhub到处传播
    	- 可以使用Dockerfile做镜像优化然后到处传播
    对比虚拟机
    	- 虚拟机太重，启动太慢，文件互通需要vm waretools支持，而且网络我总是配不明白
     对比Wsl
    	 - wsl对gui支持差
    	 - wsl还是没能解决环境的隔离问题

    	那么这么好用的工具，去哪里下载呢？度娘和谷爹会告诉你一切。
    	 那么这么好用的工具，具体来说怎么用呢？很多人做过相关的教程，b站会告诉你一切。
    	 本文只会涉及一些有难度的用法。
    	 比如，网络问题：
    		 我少的这一块dockerhub谁给我补啊！
    	方案一：
    			镜像源加速。
    	 方案二：
    			 魔法。
    			 针对魔法不生效的情况（应该是生效的，windows的 docker desktop会自动使用魔法）
    		打开魔法的TUN模式，简单来说，大部分软件默认是不走代理的（没想到吧！也就浏览器会走走代理），是否使用系统代理，这是软件自己决定的。我们可以通过一些软件的设置让它使用代理，或者是---强制爱。
    		TUN会模拟出一张虚拟网卡，所有流量都会经过这张网卡，愿不愿意，可由不得你了.....

## 让我们搞开发

## Docker Server

如果使用本地的 docker，基本可以跳过 docker Server 相关的内容。

    DS,启动！
     首先，打开windows docker desktop的这个选项：

![img](../pics/Pasted%20image%2020250327213917.png)

     这是什么意思啊啊啊？

     简单来说，让咱们得docker server在本地的127.0.0.1上暴露出2375这个端口供docker client访问，没有做TLS加密。这样，只要能够让别的机器访问到windows的localhost的2375就可以使用docker服务了。这种事情，真的有人做的到吗？
     有的有的，这种事情我们有两种方案做到。
    	方案1： 使用端口转发
    			把localhost的端口转发到我们客户端能访问到的子网上。仅仅提供思路，我自己这么做没成功过。
    	 方案2： 使用SSH转发端口

```bash
ssh -L localhost:2375:localhost user@windowServerIp
```

    		 就是利用ssh把本地的localhost和docker server主机上的localhost的2375做了双向转发。可以使用autossh自动维持这个转发

```bash
autossh -M 20001 -f -N -L 2375:loalhost:2375 user@windowsServerIp
```

    可以使用秘钥和ssh config来简化连接，不用每次都要输密码，这里不展开。

    然后打开我们自己机器上的docker,也许你已经可以连接上docker server了，使用

```bash
	docker ps
```

    看看有没有docker server上的容器。
    没有的话，对它使用docker context吧
     创建一个Docker Context并使用，它告诉了我们的docker客户端连接哪个服务器。

```bash
docker context create context_name --dcoker="host=tcp://localhost:2375"
docker context use contex_name
```

    现在应该能连接到windows docker server了，如果还是做不到的话，
     问问神奇的DeepSeek吧.

    还有一件事，在windows上开放2375的出站和入站（个人认为使用SSH的方式其实不需要这哈），当然你要是连22端口的也没开放导致SSH连接不上的话，去好好学习一下SSH吧。

## 一个简单的 ubuntu 开发环境

### 使用 VsCode 连接到容器

首先起一个 ubuntu22.04 的容器

```
docker run --name uun -itd ubuntu:22.04
```

在 VsCode 的远程开发栏找到开发容器
![](../pics/Pasted%20image%2020250327214039.png)

进入这个容器

然后，呼出命令行，你可以像在本地使用 Linux 一样使用这个容器了。
你的代码会保存在容器里，会使用容器里的编译器编译。
关于 C++容器里的环境的搭建和推荐的容器会在后面说，熟悉的同学可以自己开始动手了。

下面的内容仅仅适用于希望在 VSCode 上使用 neovim 的同学

## neovim 适配

1. 下载 VSCode Neovim 插件，这个插件把 VsCode 的编辑器替换成原生的 neovim 了
2. 本地有自己的 neovim 配置，最好为一些不想再 VsCode 中启用的插件增加检查逻辑：如果检查到在 vscode 理启动，就不启动插件
3. 对于 neovim 无限加载问题的解决方案（打开太多窗口或使用 copliot 会出现）
   1. 本地下载 0.9.4 版本的 neovim
   2. 使用 1.18.9 版本的 VSCode Neovim 插件
   3. 在插件设置中设置启动本地的 0.9.4 版本的 neovim
4. 更多快捷键，组合键在 vscode 的键盘快捷设置里调整

恭喜你，现在拥有了一套分布式，前后端分离的开发环境。
