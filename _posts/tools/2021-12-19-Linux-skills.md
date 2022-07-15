---
layout: post
title: "Linux实用小技巧"
subtitle: "Common Linux Operations and Configurations"
date: 2021-12-19 11:13:52
author: "HapppyTsing"
catalog: false
header-style: text
tags:
  - Linux
---

# 一、常用命令

查看系统信息

```bash
uname -a
cat /etc/issue
lsb_release -a

# 查看当前文件夹的大小
du -sh
# 查看当前文件夹下每个子文件夹分别多大
du -h --max-depth=1

# 以人类能看懂的单位显示文件大小
ls -lh

# 查看cpu
lscpu

# 查看内存硬件
dmidecode -t memory

# 查看内存使用情况
free -g
```

wget：下载文件

```bash
wget -c http://https://github.com/xxx/example/v1.whl
```

参数：

- O：将文件下载到指定目录中
- c：断点续传，如果下载中断，那么连接恢复时会从上次断点开始下载

tar：解压

| 参数 | 解释                                                                               |
| ---- | ---------------------------------------------------------------------------------- |
| z    | 通过 gzip 支持的压缩或解压缩。还有其他的压缩或解压缩方式，比如 j 表示 bzip2 的方式 |
| x    | 解压缩                                                                             |
| v    | 在压缩或解压缩过程中显示正在处理的文件名                                           |
| f    | f 后面必须跟上要处理的文件名。                                                     |

ssh：远程连接

```bash
ssh user@ip
# ssh root@47.108.147.2
```

scp：上传/下载文件

```bash
# 1. 上传
scp  /path/filename  username@serverIp:/path

# 2. 下载
scp ubuntu@101.43.55.140:/path/filename   ~/local_dir（本地目录）
```

如何避免重复输入密码：

```bash
# 1. 查看是否存在秘钥，若存在，则跳过下面的步骤
ls -al ~/.ssh

# 2. 创建秘钥对
ssh-keygen -t ed25519 -C "your_email@example.com"
# 注：如果您使用的是不支持 Ed25519 算法的旧系统，请使用以下命令：
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# 回车三次

cd ~/.ssh
cat id_ed25519.pub
# 复制

# 3. 登录远程主机，并粘贴公钥到~/.ssh/authorized_keys
ssh ubuntu@101.43.55.140
vim ~/.ssh/authorized_keys
# 粘贴
```

# 二、环境变量

**1. 读取环境变量**

- `export`：显示所有环境变量
- `echo $name`：输出某个具体环境变量的值，比如`echo $PATH`

**2. 设置环境变量**

- **export name=value**
  - 生效时间：立即生效
  - 生效期限：当前终端有效，窗口关闭后无效
  - 生效范围：仅针对当前用户有效
- **~/.bashrc**

  ```
  vim ~/.bashrc
  export PATH=$PATH:/new/path
  ```

  - 生效时间：交互式、non-login 方式进入 bash 运行时就会生效，或者手动`source ~/.bashrc`生效
  - 生效期限：永久有效
  - 生效范围：仅针对当前用户有效

- **~/.bash_profile**
  - 生效时间：交互式、login 方式进入 bash 运行，执行一次。或者手动 source。通常`~/.bash_profile`会调用`~/.bashrc`
  - 生效期限：永久有效
  - 生效范围：仅针对当前用户有效
- **~/.profile**
  - 感觉和.bash_profile 类似，如果使用 zsh 的话，修改.profile 即可。
- **/etc/bashrc、/etc/profile**
  - 生效范围：对所有用户有效

# 三、ubuntu shell 翻墙

**1. 查看是否已经翻墙**

```bash
curl -v https://www.google.com
```

**2. 下载安装**

```bash
cd ~ && mkdir clash && cd clash
wget https://github.com/Dreamacro/clash/releases/download/v1.8.0/clash-linux-amd64-v1.8.0.gz
gzip -d clash-linux-amd64-v1.8.0.gz
mv clash-linux-amd64-v1.8.0 clash  # 重命名为clash
chmod +x clash
```

**3. 获取[config.yaml](https://yingyun-g.pw/doc#/Advanced/Clash)**

```bash
cd ~/clash
# 直接在mac的clash中复制过去即可，无需下载。
wget -O config.yaml https://yingyun-rss.com/link/wkHLnZKJUn6hnYAY?clash=1
```

此时 `~/clash` 文件夹下有两个文件：

- 可执行文件：clash
- 配置文件：config.yaml

**4. 运行 clash： `-d` 指定配置所在的文件夹，本次指定为当前文件夹 `~/clash`**

```bash
cd ~/clash && ./clash -d .
```

此时会多出两个文件：

- 缓存文件：cache.db
- Country.mmdb

```bash
wget -O Country.mmdb [https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@release/Country.mmdb](https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@release/Country.mmdb)
```

**5. 设置命令行代理**

```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

- 临时修改：将上述命令运行即可
- 永久修改：将其写入配置文件`~/.zshrc`中

此时，已经可以正常使用，但为了精益求精，接下来需要解决两个问题：

- [如何开机默认启动？](https://github.com/Dreamacro/clash/wiki/clash-as-a-daemon)
- [如何切换代理？](https://clash.gitbook.io/doc/restful-api/proxies)默认使用的是 `config.yaml → proxies 中的第一个`

**6. 开机默认启动：clash as a daemon**

移动文件 `~/clash` 中的文件到指定位置：

```bash
cd ~/clash
sudo mkdir /etc/clash/
sudo cp clash /usr/local/bin
sudo cp config.yaml /etc/clash/
sudo cp Country.mmdb /etc/clash/

# 清理无用文件
rm -rf ~/clash
```

Create the systemd configuration file at `/etc/systemd/system/clash.service`：

```bash
sudo vim /etc/systemd/system/clash.service
[Unit]
Description=Clash daemon, A rule-based proxy in Go.
After=network.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/clash -d /etc/clash

[Install]
WantedBy=multi-user.target
```

设置在系统启动时启动 clash：

```bash
sudo systemctl enable clash
```

立即启动、重启 clash：

```bash
sudo systemctl start clash
sudo systemctl restart clash
```

使用以下命令检查 Clash 的运行状态和日志：

```bash
systemctl status clash
journalctl -xe
```

**7. 切换代理：RESTful API**

> 网页：[http://clash.razord.top/#/proxies](http://clash.razord.top/#/proxies)

首先，检查并编辑 `/etc/clash/config.yaml`

```bash
# 监听在 127.0.0.1 的 9090 端口
external-controller = 127.0.0.1:9090

# 你可以加入 secret 进行 API 鉴权
secret = 081008
# 鉴权的方式为在 Http Header 中加入 Authorization: Bearer ${secret}
curl -H "Authorization: Bearer 081008"

# 如有修改，重启：
systemctl restart clash
```

> 💡 为了方便，我们不设置 secret

如下 `/etc/clash/config.yaml`中的部分设置所示， Proxy 中记录了可选择的服务器列表：

```yaml
-
  name: Proxy
  type: select
  proxies:
    - '新加坡SG'
		- '香港HKT'
    - '台湾TW'
```

如果我们也可以通过 API 知道 Proxy 中可选哪几个服务器，以及目前正选择的是那个服务器：

```bash
curl -i -X GET  http://127.0.0.1:9090/proxies/Proxy
```

如果要选择其他服务器：

```bash
curl -X PUT http://127.0.0.1:9090/proxies/Proxy --data "{\"name\":\"新加坡SG\"}"
```

> 此外，也可以通过直接将 mac 上选择好的配置 config.yaml 直接复制到对应的 linux 的 clash 的配置文件处，启动即可。

**8. 配置 ubuntu 网络代理**

设置-网络-网络代理

选择手动（manual）

HTTP 代理：127.0.0.1:7890

Socks 主机：127.0.0.1:7891

# 四、烧录 iso

**1. 下载 iso**（[中科大源](http://mirrors.ustc.edu.cn/ubuntu-releases/16.04/)）

**2. 格式化 u 盘**

```bash
# 查看挂载的u盘，一般是 /dev/sdd1、/dev/sdb
df -h
sudo fdisk -l  # 更详细

# 格式化之前，先卸载u盘
umount /dev/sdd1

# 格式化，三种格式，选择其中一种（一般第三种）
sudo mkfs.ntfs /dev/sdd1
sudo mkfs.ext4 /dev/sdd1
sudo mkfs.vfat /dev/sdd1
```

**3. 制作启动文件**

```bash
sudo dd if=/path/to/ubuntu.isoi of=/dev/sdd bs=4M status=progress && sync
# bs=4M 设置一次读写BYTES字节,status=progress 显示烧录进度
# 注意，of=/dev/sdd，而不需要加上分区号（sdd1）
# sync 用于将缓存同步到u盘中
```

# 五、文件分区

| 目录  | 建议大小                                       | 格式     | 描述                                                                                                      |
| ----- | ---------------------------------------------- | -------- | --------------------------------------------------------------------------------------------------------- |
| /boot | 1G 左右                                        | ext4     | 空间起始位置   分区格式为 ext4 /boot 建议：应该大于 400MB 或 1GB Linux 的内核及引导系统程序所需要的文件。 |
| swap  | 物理内存两倍，内存大的话，和物理内存一样大即可 | swap     | 交换空间：交换分区相当于 Windows 中的“虚拟内存”。                                                         |
| /     | 150G-200G                                      | ext4/xfs | 根目录                                                                                                    |
| /tmp  | 5G 左右                                        | ext4     | 系统的临时文件，一般系统重启不会被保存。（建立服务器需要，家庭用也可不挂载)                               |
| /home | 剩余所有                                       | ext4/xfs | 用户工作目录；个人配置文件，如个人环境变量等；所有账号分配一个工作目录。                                  |

# **参考文献**

- [Linux 环境变量配置全攻略](https://segmentfault.com/a/1190000038313883)
- [Linux 初始化 Shell 和插件](https://leqing.work/2021/04/01/Linux-Initialize-Shell-And-Plugins/)
- [Clash RESTful API](https://clash.gitbook.io/doc/restful-api)
