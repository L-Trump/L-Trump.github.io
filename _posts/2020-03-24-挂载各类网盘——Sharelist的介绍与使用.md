---
title: 挂载各类网盘——Sharelist的介绍与使用
tags:
  - NAS
  - Docker
---

# 前言 

最近在维护自己的Scoop仓库，然而Scoop的软件下载地址需要是直链，直接找直链网盘又太贵，于是便有了这篇文章

# 关于Sharelist

Github项目地址：[Sharelist](https://github.com/reruin/sharelist)

这是一个可以挂载网盘的私人网盘，以我自己搭建的Sharelist为例：

![Snipaste_2020-03-24_09-30-27.png](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/Snipaste_2020-03-24_09-30-27.png)

同时挂载的网盘里的文件可以直接用直链的方式下载

# Sharelist的安装

## 本地安装

```bash
git clone https://github.com/reruin/sharelist.git
cd sharelist
bash ./install.sh
```
## 远程安装

```bash
wget --no-check-certificate -qO-  https://raw.githubusercontent.com/reruin/sharelist/master/netinstall.sh | bash
```

## 更新

```bash
bash update.sh
```

## 使用Docker


由于群晖的DSM蛋疼的阉割了包管理器，所以只能走Docker途径

然后因为DockerHub上的sharelist镜像版本很老了，所以推荐自己构建：

可以直接使用docker-compose，记得添加自己要用的目录：

```bash
docker-compose up
```

当然也可以手动构建：

```bash
git clone https://github.com/reruin/sharelist.git
cd sharelist
sudo docker build -t username/sharelist . //构建docker镜像，最后的那个点别忘了
```

中间npm安装的时候有红字警告不要慌，等着就好

等待十分钟左右，Docker镜像就构建完成了，然后我们可以直接从群晖的Docker GUI里面启动镜像并配置，也可以用docker命令启动：

```bash
docker run -d -v /docker/sharelist/cache:/sharelist/cache -p 33001:33001 --name="sharelist" username/sharelist
```

这里的`/sharelist/cache`是容器内sharelist的用户配置文件目录，把这个链接到容器外的话能在删除容器更新的时候保留配置

Docker镜像更新：

```bash
cd sharelist
git pull
sudo docker build -t username/sharelist . 
```

# Sharelist的使用

## 基础配置

安装完后打开http://ip:33001就会跳出来配置页面：

![Snipaste_2020-03-24_10-09-10.png](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/Snipaste_2020-03-24_10-09-10.png)

虚拟路径默认是用了Sharelist自带的演示样例，详见[Example](https://github.com/reruin/sharelist/tree/master/example)

此时对于文件的挂载，可以直接在这里进行添加（或者日后进管理界面），当然我并不建议这种做法，我们先来介绍一下Sharelist的一个逆天功能：

## Fs虚拟目录

FileSystem功能本来只是用来挂载本地文件的，然而配合虚拟目录就直接无敌了，具体操作如下

- 首先先挂载一个和容器外联通的目录，以FilesSystem挂载：

  ![Snipaste_2020-03-24_10-23-53.png](https://xqhma.oss-cn-hangzhou.aliyuncs.com/image/Snipaste_2020-03-24_10-23-53.png)
  
  挂载完后保存
  
- 在主机中找到这个目录，这时你可以直接放文件/文件夹，即挂载本地文件

  然后骚操作就来了，在这个目录下你还可以创建一个以`.d.ln`为后缀的文件名，文件内容是
  
  ```bash
  挂载标识:挂载内容
  ```
  
  这样就在这个目录下建立了等效的一个虚拟挂载目录，挂载了什么呢？后面细说
  
这样的优点其实很明显，修改方便，备份简单

## 挂载Onedrive

Onedrive有很多的挂载形式

### 1、使用分享ID挂载

- 标识：od
- 内容：分享的文件ID
- 示例：od:s!Apo33BTbGqqHhx3q6Gtb62WI6p59

**使用分享ID挂载的形式最多只支持30条显示结果**

### 2、使用API挂载

- 标识：oda
- 内容：
  - 完整：OneDrive路径->应用ID\|应用机钥\|回调地址\|refresh_token
  - 指定目录向导：OneDrive路径
  - 向导模式：/
- 示例：oda:/

建议使用向导模式，需要注意的是，由于微软的新的安全策略，无HTTPS的网站无法直接被指定为回调地址，因此回调地址可使用中转地址：

[https://reruin.github.io/redirect/onedrive.html](https://reruin.github.io/redirect/onedrive.html)

### 3、链接挂载OneDrive For Business

- 标识：odb
- 内容：分享链接
- 示例：odb:https://caomsacid0-my.sharepoint.com/:f:/g/personal/sunziyang97_caoms_ac_id/Eq3WZR7u8q1LnIDa3miPQOoB1yqrkZEwBjYYcRsS8dGgtg?e=t4MufS

### 4、挂载世纪互联版Onedrive

- 标识：odc
- 内容：
  - 完整：//应用ID/路径?client_secret=应用机钥&redirect_uri=回调地址&refresh_token=refresh_token&tenant=组织名
  - 向导模式：/
- 示例：odc:/

回调地址同样应使用中转地址，组织名为https://\*\*\*-my.sharepoint.cn/的\*\*\*部分

## 挂载Google Drive

### 1、通过分享ID挂载

- 标识：gd
- 内容：分享的ID
- 示例：gd:1cEA4umECe_-7aqBvq44AiPYxQ95zP8jr

### 2、通过API挂载

- 标识：gda
- 内容：
  - 完整：//应用ID/root?client_secret=应用机钥&redirect_uri=回调地址&refresh_token=refresh_token   
  - 向导模式：/
- 示例：gda:/

建议使用向导模式

## 挂载蓝奏云

- 标识：lanzou
- 内容：密码@分享目录ID
- 示例：
  - lanzou:b471209
  - lanzou:b0bw9myva@9234

## 挂载天翼云盘

### 账号密码挂载（推荐）

- 标识：ctcc
- 内容：
  - 完整：//用户名/文件夹ID?password=密码
  - 向导模式：/
- 示例：ctcc:/

### API挂载

- 标识：ctc
- 内容：
  - 完整：//应用ID/初始文件夹ID?app_secret=应用机钥&redirect_uri=回调地址&access_token=access_token   
  - 向导模式：/
- 示例：ctc:/

均建议使用向导模式进行操作

## 挂载本地文件

- 标识：fs
- 内容：文件路径
- 示例：fs:/mnt/video2

这里可以套娃，可以做虚拟目录里的虚拟目录

## 挂载Github

这是个新加的功能，目前只能浏览

- 标识：github
- 内容：
  - 用户名
  - 用户名/Repo
- 示例：
  - github:L-Trump
  - github:L-Trump/scoop-raresoft

## 挂载h5ai

h5ai是另一个轻型个人网盘，套娃开始了

- 标识：h5ai
- 内容：地址
- 示例：h5ai:https://larsjung.de/h5ai/demo/

## 挂载WebDAV

- 标识：webdav
- 内容：http://用户名:密码@地址:端口/路径?参数
- 示例：
  - https://webdavserver.com:1222/path   
  - https://username:password@webdavserver.com:1222/path   
  - https://username:password@webdavserver.com:1222/?acceptRanges=none

## 目录加密

可以在需要加密的目录下新建`.passwd`文件，文件内容如下：

```yaml
type: basic 
data: 
  - user1:password1 
  - user2:aaaaaa 
```

**YAML格式请勿胡乱修改空格**，data中为用户名:密码

## 流量中转

通过中转方式下载网盘内容，在后台管理中可以通过`中转（包括预览）`项开关

`中转路径`可指定中转生效的路径（含子路径），如只中转Onedrive，留空则全局生效

**注意，由于功能限制，以下方式强制使用中转模式：**

`OneDrive For Business`、`GoogleDriveAPI`、`Google Drive`

## 忽略文件类型

后台管理，常规设置，```忽略文件类型```可定义忽略的文件类型。例如忽略图片：```jpg,png,gif,webp,bmp,jpeg```。  

## 显示README

后台管理，常规设置，将```显示README.md内容```设为启用，当前目录包含```README.md```时，将自动显示在页面。

## 文件预览

后台管理，常规设置，将```详情预览```设为启用即可对特定文件进行预览。目前支持：   

### 文档类

由[preview.document](plugins/drive.document.js)插件实现，可预览md、word、ppt、excel。

### 多媒体

由[preview.media](plugins/drive.media.js)插件实现，可预览图片、音频、视频提供。   
后台管理，插件设置，```支持预览的视频后缀```可定义可预览视频类型。  

### Torrent  

由[preview.torrent](plugins/drive.torrent.js)插件实现，为种子文件提供在线预览。  

## 文件目录上传

在登录状态（页面顶部会出现上传按钮），可向 本地磁盘(fs)、OneDriveAPI(oda)、GoogleDriveAPI(gda) 上传文件/目录。  
目前处于实验性阶段，可能出现各类异常。    

## WebDAV导出

可将挂载源以WebDAV方式转出，目前支持列表、下载功能。可在 后台管理->常规设置里 设置webDAV路径。  

## 下载链接有效期

后台管理，常规设置，设置```下载链接有效期```后，下载链接将在此时间段内有效。若要关闭此功能，请设置为0。     

## Nginx(Caddy)反代注意事项

使用反代时，请添加以下配置。  
Nginx   
```ini
  proxy_set_header Host  $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;

  proxy_set_header Range $http_range;
  proxy_set_header If-Range $http_if_range;
  proxy_no_cache $http_range $http_if_range;
```   
如果使用上传功能，请调整 nginx 上传文件大小限制。   
```ini
  client_max_body_size 8000m;
```   
Caddy   
```ini
  header_upstream Host {host}
  header_upstream X-Real-IP {remote}
  header_upstream X-Forwarded-For {remote}
  header_upstream X-Forwarded-Proto {scheme}
```
