---
title: Ubuntu下搭建前端开发环境和常用工具
date: 2017-01-21 15:52:25
categories: 前端
banner: http://img.yanyuanfe.cn/isodesk.jpg
tags:
	- Linux
---


> 现在越来越多的公司都要求开发熟悉Linux环境下开发，常用的Linux版本为Ubuntu，在Ubuntu下开发需要的一些环境和工具是必然要了解的。

![image](http://img.yanyuanfe.cn/isodesk.jpg)

<!--more-->


<div class="tip">
    距离刚开始入职实习到现在已经一个月，还记得刚领到公司配的电脑，打开发现是Ubuntu，整个人都是懵逼的，之前虽然在虚拟机跑过Ubuntu，但是也还没有完全使用它来开发，所以还是吃惊不小，不过心里也有些激动，毕竟Ubuntu要是用的6也挺装逼的不是吗？后来经过一周的熟悉，也掌握了一些基本的操作，也曾踩过许多坑，所以写下这篇文章来总结一下。
</div>


### 科学上网

科学上网应该是每个开发者的必备技能吧，在一台新电脑上，上不了Google简直浑身难受。而且科学上网后上Github的速度都是杠杠的。

目前所知科学上网的几种姿势里，蓝灯最简单方便，shadowsocks最好用，下面分别讲解下其安装方法：

PS：Ubuntu下最常用的快捷键是打开终端：Ctrl+Alt+T。
Win键：快速打开Dash Home应用搜索。

- Lantern：蓝灯。
蓝灯的Github：https://github.com/getlantern/lantern
进去直接下载Ubuntu的deb安装包，下载后双击打开按提示安装即可。
安装完成后可以按win键进入Dash Home搜索lantern找到蓝灯，或者在命令行输入lantern即可自动打开浏览器，看下是不是能上谷歌了。
- shadowsocks
shadowsocks应该是现在最稳定可靠的科学上网利器了，在Ubuntu下安装和配置我也是捣鼓了好几天。
在Ubuntu下使用shadowsocks最好安装其GUI图形客户端——shadowsocks-qt5，这样以后每次操作会很方便。

shadowsocks-qt5的安装指南：https://github.com/carvenli/shadowsocks-qt5-wiki

**Ubuntu**

通过PPA源安装，仅支持Ubuntu 14.04或更高版本。


``` bash
$ sudo add-apt-repository ppa:hzwhuang/ss-qt5
$ sudo apt-get update
$ sudo apt-get install shadowsocks-qt5
```
我在安装的时候就出问题了，在运行sudo apt-get update的时候报错hash校验和不符，我百度谷歌试了N种解决方案都没用，甚至重装了一遍系统也一样，最后在同事电脑上安装也不行，后面发现好像是系统问题，于是我在自己的Win10笔记本的虚拟机上试了下，16.06版本的居然可以安装，而公司电脑默认安装的都是Ubuntu14.04的版本的，后来我果断装了Ubuntu16.06才解决这个问题。

shadowsocks不是像lantern那样安装了就能用的，它需要你有服务器节点，在客户端界面进行相关配置，具体资源需要自己去发掘了，最简便的是导入配置文件就好了，然后连接节点，这时候还不能科学上网，Ubuntu需要配置Sockt5代理，打开设置>网络>代理，选择手动，设置Socket5，地址一般是127.0.0.1，端口为8080。然后确定输入密码就可以尽情上Google了。


<div class="tip">
Ubuntu16.04的软件中心应该是有bug，安装不了第三方.deb文件，我们只有使用dpkg -i 或者gdebi的方式安装，我使用的是后者，因为后者功能更加强大。要使用gdebi命令先要安装它：
打开终端并使用下面的命令:


 sudo apt-get install gdebi


然后就可以安装.deb文件了。安装过程如下：先切换到你下载的lantern的安装文件目录下，直接使用：


 sudo gdebi lantern-installer-beta-64-bit.deb

</div>


### Chrome

Chrome是我钟爱的浏览器，没有之一。新电脑首先安装的软件就是Chrome，安装Chrome直接进入百度搜索Chrome进入官网下载deb包安装即可。

### Sublime

终于轮到开发工具了，Sublime应该是最轻量级的编辑器了吧，她最大的特点就是秒开，配合其强大的插件库也是如有神助。
进入Sublime的官网：http://www.sublimetext.com/，可直接下载其deb安装包。
当然，你也可以通过终端，仅需三行命令：

``` bash
$ sudo add-apt-repository ppa:webupd8team/sublime-text-3

$ sudo apt-get update

$ sudo apt-get install sublime-text-installer

```
最后可在Dash Home中见到Sublime-text的软件图标，点击就可使用了.
你也可以从命令行启动：

``` bash
$ subl
```


<div class="tip">
    在使用Sublime时遇到的问题：首先我们都会去安装自己常用的插件来提高自己的开发效率，在安装packge的时候经常遇到安装失败的问题，因为Sublime Text的很多package repository都在托管在github上，但是github在国内的网络环境下有时……。因此在使用Package Control安装插件时，会出现下面的Prompt：
    ![image](http://img.yanyuanfe.cn/130046_LhBu_243155.png)
    解决方案如下：  
    1.  命令行输入：

```bash
$ dig @8.8.8.8 -t A sublime.wbond.net +noall +answer
```

输出如下：  

  ; <<>> DiG 9.9.5-3-Ubuntu <<>> @8.8.8.8 -t A   
  sublime.wbond.net +noall +answer  
  ; (1 server found)  
  ;; global options: +cmd  
  sublime.wbond.net.  82  IN  A   50.116.34.243    
  那么IPv4 地址就是50.116.34.243.   
  2.  命令行输入

``` bash
$ sudo vim /etc/hosts
```

打开 /etc/hosts文件，然后用上面的IPV4地址将{}替换掉就OK了。  
{IPv4 address}    sublime.wbond.net 

</div>


### WebStorm
WebStorm被成为前端开发神器，我在项目开发的过程中使用的也是这个IDE，值得一提的是它的Git工具，很强大，特别是在处理代码冲突的时候，你会被它折服的。

**安装**

下载：http://www.jetbrains.com/webstorm/

解压下载的 gz 包，命令行下运行：

``` bash
$ cd bin
$ ./webstorm.sh
```
然后就安装成功了，不过 WebStorm 默认情况下是需要收费的，你可以去找找密钥试试。

### Node
Node也应该是前端工程化必备的一个工具了。
安装Node先安装nvm。
nvm 的全称是 Node Version Manager，之所以需要这个工具，是因为 Node.js 的各种特性都没有稳定下来，所以我们经常由于老项目或尝新的原因，需要切换各种版本。
**安装**
``` bash
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```
**配置**
``` bash
$ export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```

安装完成后，你的 shell 里面应该就有个 nvm 命令了，调用它试试。

``` bash
$ nvm
```

当看到有输出时，则 nvm 安装成功。

使用 nvm 的命令安装 Node.js 最新版：

``` bash
$ nvm install node
```

然后在任何新的shell只是使用已安装的版本：

``` bash
$ nvm use node
```

安装特定版本Node：

``` bash
$ nvm run node --version
```

安装完成后，查看一下


``` bash
$ nvm ls
```
这时候可以看到自己安装的所有 Node.js 版本，那个绿色小箭头的意思就是现在正在使用的版本。
如果你那里没有出现绿色小箭头的话，告诉 nvm 你要使用 7.4.0 版本

``` bash
$ nvm use 7.4.0
```

然后再次查看，这时候小箭头应该出现了。

OK，我们在终端中输入


``` bash
$ node
```

REPL(read–eval–print loop) 应该就出来了，那我们就成功了。

随便敲两行命令玩玩吧。


### Git

Git是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。通过使用git工具，我们可以实现团队间合作开发统一管理，可以从远程仓库中提取代码，也可以把代码上传到远程仓库，从而实现代码的同步更新。
**安装**

``` bash
$ sudo apt-get install git
```
配置git用户名和邮箱，之后就可以使用git工具了。

``` bash
$ git config --global user.name  "用户名或者用户ID"
```
``` bash
$ git config --global user.email  "邮箱"
```


### zsh
一开始我不知道还有zsh这个东东的，后来看到同事的终端怎么比我的漂亮，就去琢磨了下，原来这就是zsh——传说中的终极Shell。
Linux发行版通常默认的Shell就是Bash。也就是你刚开始打开的终端。

Bash确实是不错的Shell，但仍有用很多不尽人意的地方，如自动补全的功能不够强大，定位较长路径不够方便，命令历史管理不够完善等。

使用zsh，你会变得很Geek。

想知道你的系统有几种shell，可以通过以下命令查看：

``` bash
$ cat /etc/shells
```
**安装zsh**


``` bash
$ sudo apt-get install zsh
```
安装完成后设置当前用户使用 zsh：

``` bash
$ chsh -s /bin/zsh
```

根据提示输入当前用户的密码然后**重启**就可以了（一开始安装了没重启就用发现还是Bash还以为安装错了）。

**安装oh my zsh**

因为 zsh 的默认配置及其复杂繁琐,让人望而却步,直到有了oh-my-zsh这个开源项目,让zsh配置降到0门槛.而且它完全兼容 bash .

地址:https://github.com/robbyrussell/oh-my-zsh

它就是为配置你的 zsh 而生的.

安装「oh my zsh」可以自动安装也可以手动安装。

自动安装：

``` bash
$ wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
```

手动安装：


``` bash
$ git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

安装 oh-my-zsh 时,它自动读取你的环境变量并且自动帮 zsh 进行设置.

### 微信

**Electronic-wechat**

**Ubuntu微信客户端下载**

开箱即用的稳定版应用：
https://github.com/geeeeeeeeek/electronic-wechat/releases 

**项目地址**

https://github.com/geeeeeeeeek/electronic-wechat

**如何使用**

在下载和运行这个项目之前，你需要在电脑上安装 Git 和 Node.js (来自 npm)。

在命令行中输入:
**下载仓库**


``` bash
$ git clone https://github.com/geeeeeeeeek/electronic-wechat.git
```

**进入仓库**


``` bash
$ cd electronic-wechat
```

**安装依赖, 运行应用**


``` bash
$ npm install && npm start
```

根据你的平台打包应用:

``` bash
$ npm run build:osx
$ npm run build:linux
$ npm run build:win
```

提示: 如果 npm install 下载缓慢，你可以使用 淘宝镜像(cnpm) 替代 npm 。

### Shutter

**Ubuntu下的截图利器Shutter**

1. 打开Ubuntu软件中心,搜索Shutter并安装
2. 搜索系统中安装的Shutter并使用
3. 体验功能齐全的截图工具吧，包括全屏幕截图，区域截图，下拉菜单截图等超多功能 

### End

到此为止，介绍了一些Ubuntu下前端开发环境的搭建和常用的工具，后续有好的工具会再次更新的。



