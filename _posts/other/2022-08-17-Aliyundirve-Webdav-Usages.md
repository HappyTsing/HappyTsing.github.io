---
layout: post
title: "玩转阿里云盘"
subtitle: "aliyundirve-webdav usages"
date: 2022-08-17 11:22:00
author: "HapppyTsing"
catalog: false
header-style: text
tags:
  - movie
---

# descriptions

[阿里云盘](https://www.aliyundrive.com/drive)不限速，但是默认只能播放 1080P，且不支持 TV 端。

# webdav

开源项目：[aliyundrive-webdav](https://github.com/messense/aliyundrive-webdav)

基于上述项目，可以选择 docker 快捷使用：

```shell
docker run -d --name=aliyundrive-webdav --restart=unless-stopped -p 8080:8080 \
  -v /etc/aliyundrive-webdav/:/etc/aliyundrive-webdav/ \
  -e REFRESH_TOKEN='your-refresh-token' \
  -e WEBDAV_AUTH_USER=admin \
  -e WEBDAV_AUTH_PASSWORD=admin \
  messense/aliyundrive-webdav
```

- REFRESH_TOKEN：阿里云盘 token，获取方法：
  - [登录阿里云盘](https://www.aliyundrive.com/drive/)
  - 控制台粘贴：`JSON.parse(localStorage.token).refresh_token`
- WEBDAV_AUTH_USER：后续登录的账号，默认即可
- WEBDAV_AUTH_PASSWORD：后续登录的密码，默认即可

docker 运行之后，在 `http://localhost:8080`输入设置的账号密码登录，即可看到阿里云盘的文件。

## usage

MAC 端推荐两个软件：

- IINA
- infuse

搭配webdav即可实现4k播放。

### IINA

复制想要播放的链接地址：`http://localhost:8080/path/to/movie.name.mkv`，进入IINA，做下图操作，点击打开即可。

![IINA](https://happytsing-figure-bed.oss-cn-hangzhou.aliyuncs.com/aliyundrive/image-20220817111638946.png)

### infuse

如下图所示，路径表示显示哪个路径的内容，输入`/`表示根目录。

![infuse](https://happytsing-figure-bed.oss-cn-hangzhou.aliyuncs.com/aliyundrive/image-20220817111801306.png)

# TV

开源项目：

- [阿里云盘TV](https://aliyunpantv.gitlab.io/)
- [KODI](https://kodi.tv/)

登录后即可播放阿里云盘中的4K视频！

> 此处也可使用webdav，但使用阿里云盘TV感觉更方便。
