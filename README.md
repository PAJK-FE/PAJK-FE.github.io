# PAJK-FE Blog
[![Build Status](https://travis-ci.org/PAJK-FE/PAJK-Blog.svg?branch=master)](https://travis-ci.org/PAJK-FE/PAJK-Blog)

基于静态博客[Hexo](https://hexo.io/zh-cn/)，[NexT](http://theme-next.iissnan.com/)主题的博客。

**Source**分支为博客源码，**Master**分支为编译生成的HTML静态文件。

依赖[Travis CI](https://travis-ci.org/)自动编译部署。

## How to publish new Post
1. 准备环境
```bash
npm install -g hexo-cli   #如果已安装可跳过
git clone https://github.com/PAJK-FE/PAJK-FE.github.io.git
cd PAJK-FE.github.io
npm install
```
拉完代码后，默认是**source**分支,如果不是，请切换到source分支后操作

2. 生成新文章
```bash
hexo new title   #title替换为文章标题
```
用编辑器打开source/_posts/title，用Markdown方式以及[NexT特有标签](http://theme-next.iissnan.com/tag-plugins.html)写文章

3. 预览文章（非必要）
```bash
hexo s -o
```
浏览器会自动打开预览页面，保存文件后，刷新页面能看到最新结果

4. 发布文章
```bash
git add .
git commit -m 'update new post'  #随意填写commit信息
git push -u origin source        #不要提交到master分支，不要提交到master分支，不要提交到master分支
```
push到source分支后，Travis CI会自动编译部署。1~2分钟后刷新博客地址，确认发布成功

**注意**:因为每次source提交都会触发Travis CI，所以请尽量完成文章后再push代码

## 常见问题
#### 如何设置阅读全文？
推荐在文章中使用`<!-- more -->`手动进行截断

#### 文章发布后可以修改吗？
可以。编辑已有文章后push即可，但是不建议修改标题和时间，这样会丢失所有评论和阅读数

#### 如何删除文章？
直接删除source/_posts下对应文件，push代码

#### 如何插入图片
todo

#### 如何给文章添加标签，分类和作者？
创建新文章后，模板如下
```
---
title: title
date: 2017-05-22 10:23:59
tags:
categories:
author:
---
```
* tags后填写标签，多标签用`;`来分割
* categories后填写分类，如填多个分类会处理为父子分类，因此不建议填写多个分类，详见[这儿](https://hexo.io/zh-cn/docs/front-matter.html)
* author后填写作者名称。目前通过修改主题简单实现，待官方支持后更改

## TODO
* 多作者支持（现采用临时方案，修改主题）待NexT5.2.0官方支持多作者

## Any issues
有任何问题请联系chengandpeng@gmail.com
