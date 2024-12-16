---
title: GitHub + PicGo 搭建个人图床
date: 2024-12-16 13:42:03
tags:
  - GitHub
  - PicGo
  - 图床
categories:
  - 胡乱捣鼓
---
# GitHub + PicGo 搭建个人图床

最近开始了写博客计划，一般写Markdown的时候一般都喜欢用Typora，但是如果不用图床的话，很多图片上传后是无法显示的，因此用GitHub和PicGo搞一个图床，还方便一点。步骤也不是很麻烦，五分钟之内就能搞好。

## 1.新建GitHub仓库

新建一个GitHub仓库，名字无所谓，自己记住就好。比如我这里创建一个名为`ImageRepo`的仓库。

## 2.创建Token

点击GitHub头像，选择`Settings->Developer settings->Personal access tokens->Tokens(classic)->Generate new token->classic`。

[![image-20241216111650090](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216111650090.png)](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216111650090.png)

将这个token复制好，一会会用到。

## 3.下载PicGo APP

[PicGo官网](https://molunerfinn.com/PicGo/)

我这里用的是macOS，下的就是dmg，根据自己的操作系统下载对应版本即可。

## 4.配置PicGo

打开刚才下好的APP，选择`图床设置->GitHub`。

![image-20241216150755514](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216150755514.png)

如果只用这一个图床，可以直接修改默认配置：

- 图床配置名：`任意`
- 设定仓库名：`username/repoName`(根据自己的仓库设置：username是你的GitHub名字，repoName是仓库名字)
- 设定分支名：`main`(如果有其他分支，根据需要设置即可)
- 设定Token：刚才从GitHub复制的token
- 设定存储路径：根据自己需要设置
- 设定自定义域名：`https://cdn.jsdelivr.net/gh/username/repoName`(username和repoName同上)

点击保存，这样就设置完成了。可以在上传区上传一张照片测试一下，如果在相册中出现，说明已经成功了。

## 5.配置Typora

打开Typora设置，按照图上进行修改：插入图片时选择上传图片；上传服务选择PicGo APP。Windows可能还需要选择一下APP的路径，大同小异。![image-20241216151227572](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216151227572.png)

---

至此配置就全部完成了，我们就可以用Typora愉快的插入图片，而不用担心图片的现实问题了。但在不使用特殊手段的情况下，GitHub图床可能对墙内用户不那么友好，请自行取舍。

> 参考：
>
> [如何使用GitHub搭建个人图床，配合typora使用](https://www.bilibili.com/opus/916418138329841670)