# 风萧古道

## 安装启动
git clone 之后，不能直接使用``hugo server -DC``启动，而应该要先安装子模块，也就是主题theme。
本项目使用了hugo-PaperMod的主题，因此需要在项目根目录下使用以下命令：

```
git submodule update --init --recursive 
```

然后启动
```
hugo server -D
```

最后访问localhost:1313 即可看到博客了。

## 发表文章
### 创建一个文章文件

```
hugo new posts/你的文章名.md
```

### 编辑文章
文章文件开头如下：

```
---
title: "文章名字" //文章名字
date: 2020-02-09T17:21:03+08:00 //发布时间
draft: false  //是否是草稿
categories: ["分类"]  //分类
tags: ["标签"] //标签
---

这边是正文
```

### 保存并查看

保存后在网站上即可看到文章