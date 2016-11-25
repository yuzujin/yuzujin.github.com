---
layout: page
title: 使用github pages 和jekll-bootstrap搭建博客
---
{% include JB/setup %}

## 创建一个github仓库
首先，登录[https://github.com](https://github.com)建一个仓库，名字为username.github.com，将username替换为你实际的用户名，必须使用你的用户名，这点很重要。

## 安装Jekll-bootstrap
执行以下几个命令

    git clone https://github.com/plusjade/jekyll-bootstrap.git username.github.com
    cd username.github.com
    rm -f .git && rm -f .gitignore
    git init
    git add .
    git remote add origin https://github.com/username/username.github.com.git
    git push origin master
之所以删除.git等文件，重新初始化仓库，是为了去掉原作者的提交信息。

## 运行Jekll
等待几分钟后，访问username.github.io即可显示初始页面。

假设你已经使用gem安装jekll在本地电脑上，否则使用以下命令安装：

```
    $ yum install ruby
    $ gem install jekyll
```

那么你可以cd到本地git仓库中，运行以下命令：

```
    $ cd username.github.com
    $ jekyll serve
```

然后打开浏览器，输入http://127.0.0.1:4000 就可以看到你的博客雏形。


## 新建post

可以使用rake命令创建post

```
    $ rake post title="Hello World"
```
rake命令会自动在_posts目录下创建一篇带有YAML段落的文章，文章名会被格式化，并带有日期。例如2016-06-12-helloworld.md

在文章的头部有如下内容:


```

---
layout: post
title: "Hello World"
date: 2016-06-08 17:00:12
categories: [jekyll,mood]
---
```

这段内容成为YAML Front Matter，Jekyll在处理这个文件时，会把它当成特殊文件处理，如果没有这段内容，Jekyll只会认为这个文件是一个静态文件，不会进行我们希望的markdown2html。

layout指明这篇文章使用哪个模板，模板文件定义在_layouts文件夹下。

categories表示这篇文章属于哪个类别，Jekyll会根据文章的类别进行归类，便于你浏览。

其实Jekyll会为每个category创建一个文件夹，然后将生成的html文件放到对应的category下. 

## 创建页面

依然是通过rake命令创建页面

Create pages easily via rake task:

```
$ rake page name="about.md"
```

Create a nested page:

```
$ rake page name="pages/about.md"
```

Create a page with a "pretty" path:

```
$ rake page name="pages/about"
this will create the file: ./pages/about/index.html

```

## 发布

添加或修改完文章后，本地提交，然后推向github。

```
$ git add .
$ git commit -m "Add new content"
$ git push origin master
```

## 个性化

------------------

### Find Themes


You can find and browse the latest official themes in the [Theme Explorer](http://themes.jekyllbootstrap.com/). The theme explorer is still a work in progress; it shows full-website previews of all available themes.

[Launch Theme Explorer](http://themes.jekyllbootstrap.com/)

Additionally, designers are free to publish their own themes as long as they are packaged appropriately. You can then use the same installation method outlined below to install the theme.

Directly browse current Theme Packages on GitHub: [https://github.com/jekyllbootstrap](https://github.com/jekyllbootstrap)


### Install Themes


Install a theme by using the rake task and passing in the theme's git url.

```
$ rake theme:install git="https://github.com/jekyllbootstrap/theme-the-program.git"
```

The installer uses git to download the Theme Package and then installs it. If you have obtained a Theme Package in another way, such as zip download, you can manually place it into your ./_theme_packages folder and then run the installer with the name of the theme.

```
$ rake theme:install name="THEME-NAME"
```

As a convenience, after the install is successful, the task will ask you if you'd like to switch to the newly installed theme. Type 'y' and enter to switch!

### Switch Themes


Once your themes are installed you can switch between them via rake task:

```
$ rake theme:switch name="the-program"
# for 0.1.0 users `rake switch_theme` still works.
```

## 配置博客

Your Jekyll-Bootstrap blog can be configured by setting options within the `./_config.yml` file. You'll note the extensive documentation provided within the file itself. Let's go over some of these in detail:


### Blog Post Permalink Format

Jekyll provides extensive customization options as to how your blog post permalinks will be generated. You can consult the format table here: [Jekyll Permalinks](http://jekyllrb.com/docs/permalinks/)

Jekyll-Bootstrap ships with the traditional default format for blogs:

```
permalink: /:categories/:year/:month/:day/:title/
```

Any category structure specified on a post will be included within the url, then the date, and finally the post title.

Play around with what url configuration works best for you.


### Set BASE_PATH

All post and page urls in Jekyll-Bootstrap expect to be prepended with BASE_PATH

The BASE_PATH configuration is for special scenarios in which your blog must run from a sub-directory. The main scenario being if your blog will be hosted on a GitHub Project Page. If you are deploying this website for one of your GitHub projects you must set BASE_PATH to the name of your GitHub project.

In any case, you can leave this blank if you have defined a CNAME for your site. More info is available at: [http://pages.github.com](http://pages.github.com)

NOTE:

When in localhost, your site will run from root "/" regardless of BASE_PATH.


### Enabling Comments

Jekyll-Bootstrap ships with widget codes that enable commenting for Disqus, Intense Debate, livefyre, and Facebook Comments.

To enable commenting for your blog you will need to have setup an account with one of these providers. In the _config.yml you should see a hash named comments as shown below:

```
# Settings for comments helper
# Set 'provider' to the comment provider you want to use.
# Set 'provider' to false to turn commenting off globally.
#
comments :
  provider : disqus
  disqus :
    short_name : jekyllbootstrap
  livefyre :
    site_id : 123
  intensedebate :
    account : 123abc
  facebook :
    appid : 123
    num_posts: 5
    width: 580
    colorscheme: light
```

**Choose a Provider**


Set provider to the provider you intend to use. Make sure to specify your account credentials for the relevant provider within the hash named for that provider.

In the example above, the disqus provider will be used and will be provided with jekyllbootstrap as the account short_name.

**Use a Custom Provider**

To use a custom provider, set `provider: custom`. Next create a file at this path:

```
./_includes/custom/comments
```

This file will load wherever the theme has included its comments so you can inject your own widget code via this file. If you have a provider I don't know about, consider opening a GitHub Issue so I can include it!

**Disable Comments**

Set `provider: false` to disable comments globally.

Disable comments for individual pages/posts by specifying `comments: false` in the page/post YAML Front Matter:

```
---
layout: post
category : lessons
comments : false
tags : [yay]
---
```

### Enabling Analytics

Jekyll-Bootstrap ships with website analytics codes for [Google](http://google.com/analytics), and [GetClicky](http://getclicky.com/).

To enable analytics for your blog you will need to have setup an account with one of these providers. In the `_config.yml` you should see a hash named `analytics` as shown below:

```
# Settings for analytics helper
# Set 'provider' to the analytics provider you want to use.
# Set 'provider' to false to turn analytics off globally.
#        
analytics :
  provider : google
  google : 
      tracking_id : 'UA-123-12'
  getclicky :
    site_id :
```

**Choose a Provider**

Set provider to the provider you intend to use. Make sure to specify your account credentials for the relevant provider within the hash named for that provider.

In the example above, the google provider will be used and will be provided with UA-123-12 as the account tracking_id.

**Use a Custom Provider**

To use a custom provider, set `provider: custom`. Next create a file at this path:

```
./_includes/custom/analytics
```

This file will load wherever the theme has included its analytics so you can inject your own widget code via this file. If you have a provider I don't know about, consider opening a GitHub Issue so I can include it!

**Disable Analytics**

Set `provider: false` to disable analytics globally.

Disable analytics for individual pages/posts by specifying `analytics: false` in the post/page YAML Front Matter:

```
---
layout: post
category : lessons
analytics : false
tags : [yay]
---
```


