---
layout:     post
title:      "Vim 插件和入门教程"
subtitle:   "Vim Plugins And Use"
date:       2021-04-15 15:22:00
author:     "HapppyTsing"
catalog: false
header-style: text
tags:
  - Linux
  - Vim
---

# 一、Install Vimplus

❤为了确保不被拒绝连接，设置hosts文件：

```shell
sudo nano /etc/hosts
## 将下面的内容添加上去：
# GitHub Start
151.101.64.133 raw.githubusercontent.com
# GitHub End
```
安装vim：
```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt install vim
```

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

# 二、Use Vim

vim自带教程，命令行输入：vimtutor

**操作符：**

- d：删除
- c：更改，删除+插入
- y：复制

**一个简短的动作列表：**

- w - 从当前光标当前位置直到下一个单词起始处，跳过所有空格。
- e - 从当前光标当前位置直到下一个有效单词末尾，不包括空格。
- $ - 从当前光标当前位置直到当前行末。

**删除粘贴：**

- u撤销一次，U恢复到初始状态

- ctrl-R：redo，取消撤销dd

- dd删除一行后，光标移动到准备**置入的位置的上方**，使用p粘贴

**插入命令：**

- i，光标前方插入
- a，光标后方插入
- o，光标上新开一行
- O，光标下新开一行
- cw/ce，或者c$等操作，都可以在删除一定的东西后进入插入状态

**替换命令：替换完毕后会自动回到normal状态**

- r，光标处替换一个字符
- R，光标处开始替换，按esc退出替换

**搜索命令**

- /，正序查询，n下一个，N上一个
- ?，倒叙查询
- 查询时忽略大小写：`:set ic`，ignore case，若要禁用忽略大小写：`:set noic`

**配对命令**

- 光标置于括号，%，匹配括号
- 置于某个单词，*顺序查询相同单词，#倒叙查询

**替换命令**

- `s/old/new`，替换当前行第一个匹配串

- `s/old/new/g`，替换光标所在行的匹配串
- `#,#s/old/new/g`，#,#代表首尾两行的行号，替换这两行内的匹配串
- `%s/old/new/g`，直接替换整个文件中的每个匹配串
- `%s/old/new/gc`，会找到整个文件中的每个匹配串，并且对每个匹配串提示是否进行替换

**在vim内执行外部命令**

- 输入`:!`之后，再输入shell命令即可

**打开、保存、另存为、退出**

- `:e <path/to/file>` → 打开一个文件
- `:w` → 存盘
- `:saveas <path/to/file>` → 另存为 `<path/to/file>`
- `:x`， `ZZ` 或 `:wq` → 保存并退出 (`:x` 表示仅在需要时保存，ZZ不需要输入冒号并回车)
- `:q!` → 退出不保存 `:qa!` 强行退出所有的正在编辑的文件，就算别的文件有更改。
- `:bn` 和 `:bp` → 你可以同时打开很多文件，使用这两个命令来切换下一个或上一个文件。（陈皓注：我喜欢使用:n到下一个文件）

**选择性保存**

- v进入可视模式，使用hjkl选择所需内容，也可通过$、%、*/#、0等方式选择
- 选择完毕后按 : 字符。您将看到屏幕底部会出现 :'<,'> 。
- 现在请输入 `w TEST`，其中 TEST 是一个未被使用的文件名。确认您看到了:`'<,'>w TEST` 之后按 <回车> 键。

**提取和合并文件**

- `:r 文件路径/文件名`
- 也可以`:r shell命令`
- 插入的内容出现在光标处

**补全功能**

- `CTRL-D`，输入:e后，按CTRL-D，会显示所有e开始的命令
- `tab`，只能补全命令

# 三、References

- [Vim练级攻略](https://coolshell.cn/articles/5426.html)
- [Github Vimplus](https://github.com/chxuan/vimplus)