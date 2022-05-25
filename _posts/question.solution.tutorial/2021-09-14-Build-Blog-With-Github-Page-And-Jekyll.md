---
layout:     post
title:      "Github Page 和 Jekyll 构建博客"
subtitle:   "Build Blog With Github Page And Jekyll"
date:       2021-09-14 19:37:00
author:     "HapppyTsing"
catalog: false
header-style: text
tags:
  - Github Page
  - Jekyll
---


# 一、Github Page + Jekyll = Blog

## Install Jekyll（WSL2 Debian）

Windows10上安装WSL2 Debian，安装Jekyll：[Jekyll Docs](http://jekyllcn.com/docs/installation/)

```shell
# 依赖准备，注意一定要安装ruby-dev！
sudo apt install ruby ruby-dev rubygems nodejs
# 安装
sudo gem install jekyll
# 查看是否成功
jekyll --version
# 查看是否是最新版本
gem list jekyll
```

## Install huxpro

huxpro是Jekyll的一款主题，我们无需自己重新配置，直接拿现成的就行，这个主题非常棒：[huxpro](https://github.com/WlqFigureBed/huxpro.github.io)

```shell
# 安装bundler（一种包管理器）
gem install bundler
# Installed dependencies in the Gemfile
bundle install
# 可能需要这个依赖：sudo apt install zlib1g-dev
# 本地运行
bundle exec jekyll serve
```

> bundler：Ruby开发的包管理，只需要一个bundler install 命令，就可以下载Gemfile文件里的所有依赖，由于需要安装Jekyll的主题，主题用到了一些依赖需要用bundler安装！

### Config _config.yml

你可以通用修改 `_config.yml`文件来轻松的开始搭建自己的博客:


#### Site、SNS、Build settings

```yml
# Site settings
url: "https://WlqFigureBed.github.io"

# SNS settings      
github_username: happytsing      # 你的github账号
twitter_username: leqing666      # 你的微博账号，底部链接会自动更新的。
...

# Build settings
paginate: 10              # 一页你准备放几篇文章
```

Jekyll官方网站还有很多的参数可以调，比如设置文章的链接形式...网址：

- [Jekyll - EN](http://jekyllrb.com/) 
- [Jekyll - CN](http://jekyllcn.com/)

#### Disqus settings

博客使用[Disqus](http://disqus.com)评论系统：

- 注册帐号

- 获取Shortname：leqing-work

  ![Shortname](https://github.com/WlqFigureBed/FigureBed-one/raw/master/img/20210913201228.png)

- 配置_config.yml

```yml
# Disqus settings
disqus_username: leqing-work
```

#### Analytics settings

Todo：还没弄完！🚗

网站分析，现在支持百度统计和Google Analytics。需要去官方网站注册一下，然后将返回的code贴在下面：

```yml
# Baidu Analytics
ba_track_id: 4cc1f2d8f3067386cc5cdb626a202900

# Google Analytics
ga_track_id: 'UA-49627206-1'    # 你用Google账号去注册一个就会给你一个这样的id
ga_domain: leqing.work			# 默认的是 auto, 这里我是自定义了的域名，你如果没有自己的域名，需要改成auto。
```


#### Friends

好友链接部分。这会在全部页面显示。

设置是在 `_config.yml`文件里面的`Friends`那块，自己加吧。

```yml
# Friends
friends: [
    {
        title: "Foo Blog",
        href: "http://foo.github.io/"
    },
    {
        title: "Bar Blog",
        href: "http://bar.github.io"
    }
]
```

### Post Articles

要发表的文章一般以markdown的格式放在这里`_posts/`，你只要看看这篇模板里的文章你就立刻明白该如何设置。

文章最前面有如下的格式设定：

```yml
---
layout:     post
title:      "中国高等教育的系统性失败"
subtitle:   "The Systematic Failure of Higher Education in China"
date:       2021-01-19 12:00:00
author:     "HapppyTsing"
catalog: false
header-style: text
# header-img: "img/post-bg-2015.jpg"
tags:
  - Life
  - JAVA
---
```

在引入[Rake](https://github.com/ruby/rake)工具之后，我们可以使用命令自动生成上面的文章模板：

```shell
rake post title="中国高等教育的系统性失败" subtitle="The Systematic Failure of Higher Education in China"
```

rake工具生成文件的逻辑在于Rakefile：

```shell
# WlqFigureBed.github.io/Rakefile
post.puts "header-img: \"img/post-bg-2015.jpg\""
post.puts "header-style: text"
```

第一句语句生成的markdown，在上传后会有图片：

![header-img](https://github.com/WlqFigureBed/FigureBed-one/raw/master/img/20210913205400.png)

第二句语句生成的markdown，则不会：

![header-style](https://github.com/WlqFigureBed/FigureBed-one/raw/master/img/20210914193439.png)

## Github Page

详情可以查看[Github Page Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages)。

### Create Repository

**Steps：**

- 用于作为Github Page的仓库名字必须为：`username.github.io`
- 选择 Public 的仓库，创建成功后`git clone`下项目！
- 主页为根目录中名为index.md的文件，创建：`vim index.md`，输入hello world！
- `git push`，登录：`hppts://username.github.io`，已经可以看到网站

**Two Question：**

- 如何绑定自己的域名？
- 太丑，如何优化？：Jekyll

### Bind Domain

> 记得完成：申请域名 -> 域名备案 -> [公安备案](http://www.beian.gov.cn/portal/index.do)

教程详情参见：

- [知乎 github怎么绑定自己的域名？](https://www.zhihu.com/question/31377141)
- [Github Docs Domain For Your Site](https://docs.github.com/cn/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

在[阿里云DNS](https://dns.console.aliyun.com/?spm=5176.13329450.top-nav.dbutton.42024df5rpZ6We#/dns/setting/leqing.work)设置DNS信息，如下：

![DNS](https://github.com/WlqFigureBed/FigureBed-one/raw/master/img/20210913174516.png)

在 `WlqFigureBed.github.io/CNAME`中输入如下内容：

```text
leqing.work
```

此后Github Page会自动识别，Github Page × Jekyll 这章的图中可以看到`Custom domain`会自动识别为`leqing.work`。

### Github Page × Jekyll

将刚才的Jekyll主题的所有文件复制，粘贴到Github Page的仓库中即可！

配置_config.yml：

```yml
url: "https://WlqFigureBed.github.io"   # your host, for absolute URL
baseurl: ""                             # for example, '/blog' if your blog hosted on 'host/blog'
```

配置Github Page：[Settings -> Pages](https://github.com/WlqFigureBed/WlqFigureBed.github.io/settings/pages)

![Pages](https://github.com/WlqFigureBed/FigureBed-one/raw/master/img/20210913211410.png)

# 二、Rules For Files

[Jekyll Docs structure](https://www.jekyll.com.cn/docs/structure/)

## Article Location

文章主要存储在三个地方：

- _posts
- _includes/about

- _includes/posts

### _posts

存放真正发布的内容！

其中的文章开头有一些特殊设置，详见[Post Articles](#Post Articles)

### _includes/about

存放两个文件：

- en.md
- zh.md

是about页面的个人介绍信息

### _includes/posts

存放一系列文件夹，其中每个文件夹下存放两个文件：

- en.md
- zh.md

```shell
-- posts
	-- 2017-07-12-upgrading-eleme-to-pwa
		-- en.md
		-- zh.md
```

此时再查看_posts文件夹下的文件，找到：`2017-07-12-upgrading-eleme-to-pwa.md`

其中的完整的文件内容查看：[Available Both English And Chinese Versions](#Available Both English And Chinese Versions)。

## Picture Location

img/archive-bg：archive.html

img/about-bg：about.html

img/404-bg：

- 404.html
- offline.html

img/home-bg：_config.yml

img/avatar：_config.yml

img/webicon：

- _includes/head.html

- _layouts下的所有文件都有引用`/img/icon_wechat.png`这个文件，但是只有其中的default.html没有将其注释掉！我将其改成了引用`img/webicon/2.png`，这个暂时不知道有什么用！

# 三、Special usage

## Available Both English And Chinese Versions

#### WlqFigureBed.github.io/_posts/2017-07-12-upgrading-eleme-to-pwa.md

![2017-07-12-upgrading-eleme-to-pwa.md](https://github.com/WlqFigureBed/FigureBed-one/raw/master/img/20210914211120.png)

#### WlqFigureBed.github.io/_includes/posts/2017-07-12-upgrading-eleme-to-pwa

![posts/2017-07-12-upgrading-eleme-to-pwa](https://github.com/WlqFigureBed/FigureBed-one/raw/master/img/20210913223305.png)

注意：en.md和zh.md两个都是单纯的markdown文件，且不需要：

```text
---
layout:       post
title:        "饿了么的 PWA 升级实践"
subtitle:     "Upgrading Ele.me to Progressive Web App"
...
---
```

诸如以上的内容是不需要的！直接输入内容即可！

#### Final Effects

![中英文效果](https://github.com/WlqFigureBed/FigureBed-one/raw/master/img/20210913222916.png)

# 四、References

- [Github Hux Blog](https://github.com/WlqFigureBed/huxpro.github.io)
- [Jekyll Docs](http://jekyllcn.com/docs/installation/)
- [Github Page Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages)

