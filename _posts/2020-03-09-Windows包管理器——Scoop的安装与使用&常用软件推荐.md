---
title: 强大的Windows包管理器——Scoop的安装与使用&常用软件推荐
tags:
  - Windows
  - 实用工具
  - Scoop
---

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

和原先的规则一样，代理只对当前窗口生效，每个需要代理的Powershell窗口都需要键入这两行

### 安装

首先执行命令：

```powershell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
```

执行完后会出现如下界面：

![Snipaste_2020-03-09_15-36-15.png](https://i.loli.net/2020/03/09/iRZtPlNuFGOvx18.png)

键入Y或者A后回车即可，接下来执行命令：

```
iwr -useb get.scoop.sh | iex
```

如果出现下载错误请挂上代理，如果由于上一次安装失败导致出现已安装提示无法继续的情况，请手动删除Scoop的安装目录，默认为`C:\User\你的用户名\Scoop`。

等待绿色的成功提示出现，Scoop就已安装完毕了，可以 键入`scoop help`查看帮助菜单。

## Scoop的使用

使用其实和其余的包管理器大同小异

### 安装软件

首先先查找下看看有哪些软件，以git为例：

```
scoop search git
```

会出现类似下列信息：

```powershell
'main' bucket:
    git-annex (8.20200226)
    git-chglog (0.9.1)
    git-crypt (0.6.0-701fb8e)
    git-filter-repo (2.25.0)
    git-interactive-rebase-tool (1.2.1)
    git-istage (0.2.61)
    git-lfs (2.10.0)
    git-sizer (1.3.0)
    git-tfs (0.30)
    git-town (7.3.0)
    git-up (1.6.1)
    git-with-openssh (2.25.1.windows.1)
    git (2.25.1.windows.1)
    gitea (1.11.2)
    gitignore (0.2018.07.25)
    gitkube (0.3.0)
    gitlab-runner (12.8.0)
    gitomatic (0.2)
    gitversion (5.1.3)
    llvm (9.0.0) --> includes 'git-clang-format'
    mingit-busybox (2.25.1.windows.1)
    mingit (2.25.1.windows.1)
    psgithub (2017.01.22)
    psutils (0.2020.02.27) --> includes 'gitignore.ps1'
```

可以看到其中有git的包名，于是我们直接执行：

```powershell
scoop install git
```

就完成安装了。这样的安装方式默认是仅当前用户使用的，如果需要全局安装，加上开关-g即可：

```powershell
scoop install git -g
```

### 查看已安装软件

```powershell
scoop list
```

### 更新软件及Scoop

```powershell
scoop update
```

**注意：update操作依赖于git，所以需要先安装git**

### 卸载软件/Scoop

如卸载git

```powershell
scoop uninstall git
```

如果需要卸载Scoop本身，可直接执行

```powershell
scoop uninstall scoop
```

### 查看软件信息

可查看特定软件的版本、网站、License、配置文件、可执行文件、环境变量等信息，如git：

```powershell
scoop info git
```

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

当然，你也可以在网上寻找第三方源并添加（[这里](https://github.com/rasa/scoop-directory/blob/master/by-apps.md)）：

```powershell
scoop bucket add Ash258 'https://github.com/Ash258/Scoop-Ash258.git'
scoop update
```

需要注意的是，**添加Bucket后请*务必*执行一遍scoop update**。

### 自建Bucket与维护

*我不会，暂时也没这个需求，有需要了再加吧*

### 常用软件推荐

[Scoop上的软件推荐](https://blog.xqh.ma/_posts/2020-03-11-Scoop%E4%B8%8A%E7%9A%84%E8%BD%AF%E4%BB%B6%E6%8E%A8%E8%8D%90/)