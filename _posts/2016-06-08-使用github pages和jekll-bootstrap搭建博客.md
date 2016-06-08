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

##运行Jekll
访问username.github.io即可显示初始页面。
