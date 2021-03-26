---
layout: post
title:  "Step by step: 创建 GitHub Pages 网站(使用 Jekyll)"
date:   2021-03-24 15:00:00 +0800
categories: skill
---

在内转的空隙，使用 [GitHub Pages] 建立了自己的个人博客，记录一下其中的过程，平台：Windows 10。

详细操作请参考官网：[Creating a GitHub Pages site with Jekyll](https://docs.github.com/en/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll)

# 安装 Ruby

下载 [RubyInstaller](https://rubyinstaller.org/downloads/) 选择 Ruby+Devkit 的安装包，双击安装。

在安装结束时，去除 ridk install 的选项。

# 手工安装 MSYS2

查找 Ruby 安装目录下的 `msys64\etc\pacman.d` , 添加 China 国内的更新源：

- mirrorlist.mingw32: `Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/i686`
- mirrorlist.mingw64: `Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64`
- mirrorlist.msys: `Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/$arch`

配置 cmd 代理：
{% highlight shell %}
$ set http_proxy=http://proxy.xxx.com:port
$ set https_proxy=http://proxy.xxx.com:port

$ set http_proxy_user=username
$ set http_proxy_pass=password

$ set https_proxy_user=username
$ set https_proxy_pass=password
{% endhighlight %}

然后执行 `ridk install` 安装 MSYS2，一路 Enter 即可。

# 安装 bundler/jekyll

将 Ruby 的 gem 源替换为国内的：
{% highlight shell %}
$ gem source -a https://gems.ruby-china.com --http-proxy http://username:password@proxy.xxx.com:port --remove https://rubygems.org/
{% endhighlight %}

然后：
{% highlight shell %}
$ gem install jekyll bundler
{% endhighlight %}

# 创建 repository

在 [GitHub] 上创建 repository ，名称必须为 `<user>.github.io` 。

# 使用 jekyll 构建网站

## 初始化本地仓

{% highlight shell %}
$ git init https://github.com/<user>/<user>.github.io.git
$ jekyll new .
# Creates a Jekyll site in the current directory
{% endhighlight %}

## 修改配置文件

### Gemfile

注释掉 `gem "jekyll"`, 放开 `gem "github-pages"`.

```
# gem "jekyll"
gem "github-pages", group: :jekyll_plugins
```

### _config.yml

```
- title: 安身、立命、静心、养性
- email: ruizhi_xu@163.com
- description: >- # this means to ignore newlines until "baseurl:"
    A Father! A Test Engineer! A Programmer!
- baseurl: "" # the subpath of your site, e.g. /blog
- url: "https://xuruizhi.github.io/"
- twitter_username: xuruizhi1983
- github_username:  xuruizhi
```

然后：
{% highlight shell %}
$ bundle update
{% endhighlight %}

# 添加新帖子

只需要在 _posts 文件夹中新建 YYYY-MM-DD-NAME-OF-POST.md 文件即可完成发帖。
帖子的题头格式如下：

```
---
layout: post
title: "POST TITLE"
date: YYYY-MM-DD hh:00:00 +0800
categories: CATEGORY-1 CATEGORY-2
---
```

其中 CATEGORY 用来组织最终发布的 url 路径: `skill/2021/03/24/creating-a-github-pages-site-with-jekyll`.html

# 在本地测试网站

{% highlight shell %}
$ bundle exec jekyll serve
{% endhighlight %}

打开 http://127.0.0.1:4000/ 即可。


# 维护

{% highlight shell %}
$ bundle update github-pages
{% endhighlight %}

---

> PS：作为[维基百科]、[饭否]的早期用户，在 [Web 2.0] 大潮已经过去将近20年的今天才在互联网上有了自己的一亩三分地，真是汗颜。

[GitHub]: https://github.com/
[GitHub Pages]: https://pages.github.com/
[维基百科]: https://zh.wikipedia.org/
[饭否]: https://fanfou.com/
[Web 2.0]: https://zh.wikipedia.org/wiki/Web_2.0
