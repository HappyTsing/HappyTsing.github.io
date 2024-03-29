---
layout: post
title: "Linux 初始化 Shell 和插件"
subtitle: "Linux Initialize Shell And Plugins Introduction"
date: 2021-04-01 18:32:00
author: "HapppyTsing"
catalog: false
header-style: text
tags:
  - Linux
---

> 换源：[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

# 一、zsh

```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y zsh wget curl git autojump
cat /etc/shells # 查看所有安装的shell
chsh -s $(which zsh) #修改默认shell为zsh   或者输入chsh后回车，再输入/bin/zsh
echo $SHELL # 查看修改后的使用shell
```

重启电脑后生效！

# 二、oh-my-zsh

安装 oh-my-zsh 及其插件

```shell
# 安装oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
sh -c "$(curl -fsSL https://gitee.com/forkhub-tsing/ohmyzsh/raw/master/tools/install.sh)"

# 安装zsh-syntax-highlighting：提供命令高亮
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://gitee.com/forkhub-tsing/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 安装autosuggestions：记住你之前使用过的命令
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://gitee.com/forkhub-tsing/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

启动插件：

```shell
vim ~/.zshrc
## 找到plugins=(git)，修改为：
plugins=(git autojump zsh-syntax-highlighting zsh-autosuggestions sudo extract)
# sudo是ohmyzsh自带的插件，功能是在你输入的命令的开头添加sudo ，方法是双击Esc
# extract也是自带插件，不用再去记不同文件的解压命令，方法是extract +你要解压的文件名

# 绑定~为接受建议，在文件末尾添加如下内容：
bindkey '`' autosuggest-accept

# 使用如下shell代码一键配置
sed -i '/plugins=(git)/ c plugins=(git autojump zsh-syntax-highlighting zsh-autosuggestions sudo extract)' ~/.zshrc
if [ ! `grep -c "bindkey .* autosuggest-accept" $HOME/.zshrc` -ne "0" ];then
    sed -i '$a bindkey '"'"'`'"'"' autosuggest-accept' ~/.zshrc
fi
```

安装主题(二选一)

- powerlevel10k
- pure

**1. Powerlevel10k**

```shell
# 安装字体：https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# vim ~/.zshrc 找到ZSH_THEME，修改为：ZSH_THEME="powerlevel10k/powerlevel10k", 快速配置：
sed -i '/ZSH_THEME="/ c ZSH_THEME="powerlevel10k/powerlevel10k"' ~/.zshrc
```

**2. pure**

```sh
git clone https://gitee.com/forkhub-tsing/pure.git "$HOME/.zsh/pure"

# 参见 https://github.com/sindresorhus/pure#install，快速配置：

# 删除p10k及pure的旧配置
sed -i '/fpath.*pure/d' ~/.zshrc
sed -i '/autoload.*promptinit/d' ~/.zshrc
sed -i '/prompt pure/d' ~/.zshrc

# Set ZSH_THEME="" in your .zshrc to disable oh-my-zsh themes.
sed -i '/ZSH_THEME="/ c ZSH_THEME=""' ~/.zshrc

# 新增配置
echo 'fpath+=($HOME/.zsh/pure)' >> ~/.zshrc
echo 'autoload -U promptinit; promptinit' >> ~/.zshrc
echo 'prompt pure' >> ~/.zshrc
```

保存配置：

```shell
source ~/.zshrc
```

重启 shell，若选择了 p10k 主题，此时会进入主题设置，按引导进行即可，如果设置后不满意：

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
git clone --depth=1 https://gitee.com/forkhub-tsing/vimrc.git ~/.vim_runtime

sh ~/.vim_runtime/install_awesome_vimrc.sh
```

**二、Vimplus**

安装 Vimplus——开箱即用的 vim 插件管理工具：

```shell
git clone https://github.com/chxuan/vimplus.git ~/.vimplus
git clone https://gitee.com/forkhub-tsing/vimplus.git ~/.vimplus

cd ~/.vimplus
./install.sh
```

查看 vim 插件：

```shell
vim ~/.vimrc
# 可以看到如下内容，两个call中间的内容就是我们的插件
call plug#begin()
Plug 'preservim/NERDTree'
call plug#end()
```

该 vimplus 插件默认安装了若干插件，如果安装失败：

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

- [bat](https://github.com/sharkdp/bat)，该插件用于代替 cat，高亮显示

  - arch 安装：`yay -S bat`or`sudo pacman -S bat`
  - debian 安装：`sudo apt install bat`

- thefuck

- tig，更好用的 gitlog

- ydict，翻译

# 五、shell shortcuts

- bat 代替 cat，可以高亮显示代码

- j 文件名，可以快速跳转到某个目录，而不需要输入这个目录的前序目录
- d，可以查看已经去过的目录，然后输入对应数字进入对应目录
- 出现提示时，输入~或者方向右键应用提示
- 删除 shell 中的一行内容。首先 ctrl+a 移动到行首，再 ctrl+k 删除一行的内容
- r，重复上一条命令
- code filename 命令，使用 vscode 打开某文件
- .zshrc 中配置了 plugins=(sudo)后，连续按两下 esc 键即在命令前加上 sudo
- fuck，安装了 thefuck 后输入即可纠正错误

> ### Reference
>
> - [Github powerlevel10k](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k)
> - [iterm2 和 zsh 终端体验优化](https://bytedance.feishu.cn/docs/doccn6xXJnUTedx2roBjKAMX2Gf#)
> - [程序员内功系列-iTerm2 与 Zsh 篇](https://xiaozhou.net/learn-the-command-line-iterm-and-zsh-2017-06-23.html)
> - [程序员内功系列-常用命令行工具推荐](https://xiaozhou.net/learn-the-command-line-tools-md-2018-10-11.html)
> - [oh my zsh 插件推荐](https://hufangyun.com/2017/zsh-plugin/)
