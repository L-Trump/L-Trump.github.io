---
title: 强大的Windows包管理器——Scoop的安装与使用&常用软件推荐
tags:
  - Windows
  - 实用工具
  - Scoop
---

## 2020-3-28: 汉化版已制作完成！见[Scoop-CHS](https://github.com/L-Trump/Scoop-CHS)

## 前言

用过Linux的同学对包管理器的概念一定不陌生，无论是yum，apt还是pacman都是极好用的包管理器，绝大部分的软件以及其所需的环境变量都能一条命令搞定，相对来说，windows中软件的安装就要繁琐许多。所幸，得益于Win10引进的Powershell，属于Windows的包管理器——Scoop，它来了。

与此同时，作为Windows下的包管理器，Scoop除了能安装命令行工具外，也能够安装具有图形窗口的软件，其中甚至包括Atom，VSCode，OBS-Studio等，包括我现在用来码字的工具Typora也可以在scoop中直接安装

## 系统环境要求

接下来进入正题，安装前准备：

- 不含中文字符的用户名（绝大多数人都是符合的）

- [Powershell 3+](https://aka.ms/pscore6)
  
  - Win10是自带了的，直接运行powershell命令就行，Win7及以上的其他版本可以到这里下载
  
- [.NET Framework 4.5+](https://dotnet.microsoft.com/download)

  **Powershell版本号查询方法：在Powershell中输入`$PSVersionTable`，其中PSVersion项对应的为PowerShell版本号**

## Scoop的安装

### 自定义安装路径

Scoop默认是安装在C盘中用户主目录下的，如果C盘不够大，那就是一件非常令人头疼的事情，因此，我们需要在开始前设定好scoop的安装目录（**请在以管理员身份运行的Powershell中执行以下命令**）：

```powershell
[environment]::setEnvironmentVariable('SCOOP','D:\ScoopApps','Machine') 
$env:SCOOP='D:\ScoopApps'
```

如果你在以后使用Scoop安装软件过程中采用全局安装的方式（开关-g，后文会提到），那么还需要执行：

```powershell
[environment]::setEnvironmentVariable('SCOOP_GLOBAL','D:\GlobalScoopApps','Machine')
$env:SCOOP_GLOBAL='D:\GlobalScoopApps'
```

可以把上面的路径换成你自己的路径。

### 关于代理

由于种种原因，Scoop的安装流程有时需要代理才能够进行，通常情况下直接在Windows设置或者Internet选项中更改即可，如果依然失败，请尝试以下命令：

```powershell
$env:HTTP_PROXY="http://127.0.0.1:1080"
$env:HTTPS_PROXY="http://127.0.0.1:1080"
```

需要注意的是，代理只对当前窗口生效，每个需要代理的Powershell窗口都需要键入这两行

### 安装

首先执行命令：

```powershell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
```

执行完后会出现如下界面：

![Snipaste_2020-03-09_15-36-15.png](https://i.loli.net/2020/03/09/iRZtPlNuFGOvx18.png)

键入Y或者A后回车即可，接下来执行命令：

```powershell
iwr -useb get.scoop.sh | iex
#中文版使用：
iwr -useb https://raw.githubusercontent.com/L-Trump/Scoop-CHS/master/bin/install.ps1 | iex
```

如果出现下载错误请挂上代理，如果由于上一次安装失败导致出现已安装提示无法继续的情况，请手动删除Scoop的安装目录，默认为`C:\User\你的用户名\Scoop`。

等待绿色的成功提示出现，Scoop就已安装完毕了。

## Scoop的使用

使用其实和其余的包管理器大同小异

### 查看帮助

```powershell
scoop help
```

![help.gif](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/help.gif)

### 为Scoop设置代理

Scoop的软件源是基于Git实现的，而大部分的软件源都挂在Github上，如果不用代理访问的话可能会非常慢，而挂代理有如下几种方法

- **使用系统代理+环境变量（临时）**

  系统代理直接在设置中操作就行，环境变量见[***关于代理***](/_posts/2020-03-09-Windows包管理器-Scoop的安装与使用&常用软件推荐/#关于代理)

- **使用Scoop的设置**

  这个操作是重新打开窗口也会生效的，当然只对scoop生效：
  
  ```powershell
  scoop config proxy username:password@host:port //用户名密码可省略
  如： scoop config proxy 192.168.2.110:1080
  ```
  
  删除代理：
  
  ```powershell
  scoop config rm proxy
  ```
### 更新Scoop及软件列表

虽然Scoop有时会自动执行，但还是建议在安装应用前习惯性地执行这个命令，可以保证软件列表不是过时的。

```powershell
scoop update
```

![udpate.gif](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/udpate.gif)

**注意：update操作依赖于git，所以需要先安装git**

### 安装软件

首先先查找下看看有哪些软件，以wget为例：

```powershell
scoop search wget
```

会出现下列信息：

![search.gif](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/search.gif)

可以看到其中有wget的包名，于是我们直接执行：

```powershell
scoop install wget
```

![install.gif](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/install.gif)

就完成安装了。这样的安装方式默认是仅当前用户使用的，如果需要全局安装，加上开关-g即可：

```powershell
scoop install wget -g
```

### 查看已安装软件

```powershell
scoop list
```

![list.gif](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/list.gif)

### 卸载软件/Scoop

如卸载wget

```powershell
scoop uninstall wget
```

![uninstall.gif](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/uninstall.gif)

如果需要卸载Scoop本身，可直接执行

```powershell
scoop uninstall scoop
```

### 查看软件更新

用来查看是不是有可以更新的软件：

```powershell
scoop status
```

### 更新软件

```powershell
scoop update vim
```

这样就可以更新特定软件了，如果要更新全部软件则可以 

```powershell
scoop update *
```

### 查看软件信息

可查看特定软件的版本、网站、License、配置文件、可执行文件、环境变量等信息，如vim：

```powershell
scoop info vim
```

![info.gif](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/info.gif)

### 添加Bucket（软件源）

默认的main bucket有着诸多的限制，最致命的是不允许有GUI界面的软件进入其中，但在main之外，官方也提供了其他许多的源，可以用下列命令查看：

```powershell
scoop bucket known
```

结果如下：

```powershell
PS C:\Windows\system32> scoop bucket known
main
extras
versions
nightlies
nirsoft
php
nerd-fonts
nonportable
java
games
jetbrains
```

其中强烈推荐添加extras和versions，extras中有一大批好用的软件，包括snipaste，atom，quicklook，OBS等等，而versions中则有许多软件的beta版本，如snipaste 2以后的版本等。

添加方法如下

```powershell
scoop bucket add extras
scoop bucket add versions
scoop update
```

当然，你也可以在网上寻找第三方源并添加，如我的破解软件源（[Scoop软件源排名](https://github.com/rasa/scoop-directory/blob/master/by-stars.md)）：

```powershell
scoop bucket add raresoft 'https://github.com/l-trump/scoop-raresoft'
scoop update
```

需要注意的是，**添加Bucket后请*务必*执行一遍scoop update**。

### 移除Bucket

```powershell
scoop bucket rm raresoft
```

### 利用aria2加速Scoop下载

如果你经常安装体积较大的软件，那么建议配置aria2下载，但用默认比较稳定，自行取舍吧。

开启：

```powershell
scoop install aria2 #安装aria2
scoop config aria2-max-connection-per-server 16 #设置16线程下载
scoop config aria2-split 16 #设置16线程下载分块
scoop config aria2-min-split-size 1M #设置每个分块的最小体积
scoop config aria2-enabled true #启用aira2下载，默认安装好后就是启用的
```

线程数上限为16，如果需要更大的并行线程数，请自行编译修改限制

关闭：

```powershell
scoop config aria2-enabled false #临时关闭
scoop uninstall aria2 #卸载aria2
```

### 清除缓存

Scoop是有下载缓存的，查看下载缓存：

```powershell
scoop cache
```
删除下载缓存：

```
scoop cache rm 应用名

# 或者删除全部缓存：

scoop cache rm *
```

### 清除旧版本应用

在Scoop中，当一个应用被更新以后，其旧版本其实并不会被删除，我们可以通过以下命令删除：

```powershell
scoop cleanup 应用名
# 删除全部旧版本应用：
scoop cleanup *
```

### 自建Bucket与维护

请自行查看Scoop的[Wiki](https://github.com/lukesampson/scoop/wiki)

推荐下我自己的仓库，里面全是好东西哦***\(手动滑稽\)***：

[RareSoftware](https://github.com/l-trump/scoop-raresoft)

### 常用软件推荐

[Scoop上的软件推荐](/_posts/2020-03-11-Scoop%E4%B8%8A%E7%9A%84%E8%BD%AF%E4%BB%B6%E6%8E%A8%E8%8D%90/)