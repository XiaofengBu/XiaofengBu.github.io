---
layout: post
title:  "第一篇博客"
categories: blog
date:   2019-1-28 16:29:14
tags:  blog
author: Xiaofeng
---

* content
{:toc}


## 前言

临近春节最近工作较为轻松,抽空用github pages搭建了自己的博客,同时用第一篇博客感谢下搭建的过程参考的几位大佬。

## 参考

[github搭建个人博客](https://blog.csdn.net/xudailong_blog/article/details/78762262)

[Jekyll搭建静态博客](https://643435675.github.io/2015/02/15/create-my-blog-with-jekyll/)


## 补充说明
在参<Jekyll搭建静态博客>时遇到部分问题如下:
1. ``` ruby setup.rb ``` 报找不到文件的错误,暂未解决但是不影响后续使用
2. ``` gem install jekyll ```命令执行错误 -> ruby需要下载+devkit版本,同时可以注意ruby的版本
3. ``` jekyll serve ``` 报错 ``` Dependency Error: Yikes! It looks like you don’t have jekyll-paginate or one of its dependencies installed. In order to use Jekyll as currently configured, you’ll need to install this gem. The full error message from Ruby is: ‘cannot load such file – jekyll-paginate’ If you run into trouble, you can find helpful resources at Getting Help``` 
运行
```gem install jekyll-paginate```












