---
layout:     post
title:      "Linux 初始化 Shell 和插件"
subtitle:   "Linux Initialize Shell And Plugins"
date:       2021-04-01 18:32:00
author:     "HapppyTsing"
catalog: false
header-style: text
tags:
  - Linux


---

# 一、zsh

```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y zsh
cat /etc/shells # 查看所有安装的shell
chsh -s $(which zsh) #修改默认shell为zsh   或者输入chsh后回车，再输入/bin/zsh
echo $SHELL # 查看修改后的使用shell
```

重启电脑后生效！

# 二、oh-my-zsh

❤为了确保不被拒绝连接，设置hosts文件：

```shell
sudo nano /etc/hosts
## 将下面的内容添加上去：
# GitHub Start
151.101.64.133 raw.githubusercontent.com
# GitHub End
```

接下来安装oh-my-zsh及其插件：

```shell
# 安装oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 安装zsh-syntax-highlighting：提供命令高亮
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 安装autosuggestions：记住你之前使用过的命令
git clone git://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions

# 安装autojump
apt install autojump
```

> 注意：如果GitHub被墙无法安装oh-my-zsh，可按如下方法安装：
>
> - 进入网址：https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
>
> - 复制全部内容
>
> - ```shell
>   nano install.sh
>   # ctrl + v 粘贴脚本
>   # ctrl + s 保存文件
>   sh install.sh  # bash install.sh
>   ```
>
> - 如上，安装完毕。

安装主题powerlevel10k

```shell
# 安装字体：https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf
sudo apt install git 
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

启动插件：

```shell
nano ~/.zshrc
## 找到plugins=(git)，修改为：
plugins=(git autojump zsh-syntax-highlighting zsh-autosuggestions sudo extract)
# sudo是ohmyzsh自带的插件，功能是在你输入的命令的开头添加sudo ，方法是双击Esc
# extract也是自带插件，不用再去记不同文件的解压命令，方法是extract +你要解压的文件名

## 找到ZSH_THEME，修改为：
ZSH_THEME="powerlevel10k/powerlevel10k"

# 绑定~为接受建议，在文件末尾添加如下内容：
bindkey '·' autosuggest-accept
```

使用如下脚本快速完成设置：

```shell
sed -i '/ZSH_THEME="/ c ZSH_THEME="powerlevel10k/powerlevel10k"' ~/.zshrc 
sed -i '/plugins=(git)/ c plugins=(git autojump zsh-syntax-highlighting zsh-autosuggestions sudo extract)' ~/.zshrc 
sed -i '$a bindkey '"'"'`'"'"' autosuggest-accept' ~/.zshrc 
```

保存配置：

```shell
source ~/.zshrc
```

重启shell，此时会进入主题设置，按引导进行即可，如果设置后不满意：

```shell
# 重新配置主题
p10k configure
```

# 三、vim

两种选择：

- [vimrc](https://github.com/amix/vimrc)：推荐

- [vimplus](https://github.com/chxuan/vimplus)

**一、vimrc**

```shell
git clone --depth=1 https://github.com/amix/vimrc.git ~/.vim_runtime
sh ~/.vim_runtime/install_awesome_vimrc.sh
```

**二、Vimplus**

安装Vimplus——开箱即用的vim插件管理工具：

```shell
git clone https://github.com/chxuan/vimplus.git ~/.vimplus
cd ~/.vimplus
./install.sh
```

查看vim插件：

```shell
vim ~/.vimrc
# 可以看到如下内容，两个call中间的内容就是我们的插件
call plug#begin()
Plug 'preservim/NERDTree'
call plug#end()
```

该vimplus插件默认安装了若干插件，如果安装失败：

```shell
# 进入vim文本编辑器
vim

# 安装所有插件
:PlugInstall

# 更新插件
:PlugUpdate

# 如果你不想更新所有的插件，你可以通过添加插件的名字来更新任何插件:
:PlugUpdate NERDTree
```

# 四、others

- [bat](https://github.com/sharkdp/bat)，该插件用于代替cat，高亮显示
  - arch安装：`yay -S bat`or`sudo pacman -S bat` 
  - debian安装：`sudo apt install bat`

- thefuck

- tig，更好用的gitlog

- ydict，翻译

# 五、shell shortcuts

- bat 代替 cat，可以高亮显示代码

- j 文件名，可以快速跳转到某个目录，而不需要输入这个目录的前序目录
- d，可以查看已经去过的目录，然后输入对应数字进入对应目录
- 出现提示时，输入~或者方向右键应用提示
- 删除shell中的一行内容。首先ctrl+a移动到行首，再ctrl+k删除一行的内容
- r，重复上一条命令
- code filename 命令，使用vscode打开某文件
- .zshrc中配置了plugins=(sudo)后，连续按两下esc键即在命令前加上sudo
- fuck，安装了thefuck后输入即可纠正错误

# 六、Reference

- [Github powerlevel10k](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k)
- [iterm2和zsh终端体验优化](https://bytedance.feishu.cn/docs/doccn6xXJnUTedx2roBjKAMX2Gf#)
- [程序员内功系列-iTerm2与Zsh篇](https://xiaozhou.net/learn-the-command-line-iterm-and-zsh-2017-06-23.html)
- [程序员内功系列-常用命令行工具推荐](https://xiaozhou.net/learn-the-command-line-tools-md-2018-10-11.html)
- [oh my zsh 插件推荐](https://hufangyun.com/2017/zsh-plugin/)

