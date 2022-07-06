---
tags: []
categories: ["chaos"]
description:
summary:
draft: false
isCJKLanguage: true
date: "2022-06-08"
lastmod: "2022-06-10"
title: Picgo 配置
---

# Picgo 配置

> [!cite] 参考
> [配置手册 | PicGo](https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A)

## SMMS

注册并登录 [smms](https://sm.ms/home/apitoken) 后台获取 token 值。

![](https://cdn.jsdelivr.net/gh/Molunerfinn/test/picgo/20200307182127.png)

## Github

**1.** 首先你得有一个 GitHub 账号。注册 GitHub 就不用我多言。

**2.** 新建一个仓库

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/create_new_repo.png)

记下你取的仓库名。

**3.** 生成一个 token 用于 PicGo 操作你的仓库：

访问：https://github.com/settings/tokens

然后点击 `Generate new token`。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/generate_new_token.png)

把 repo 的勾打上即可。然后翻到页面最底部，点击 `Generate token` 的绿色按钮生成 token。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/20180508210435.png)

**注意：** 这个 token 生成后只会显示一次！你要把这个 token 复制一下存到其他地方以备以后要用。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/copy_token.png)

**4.** 配置 PicGo

**注意：** 仓库名的格式是 `用户名/仓库`，比如我创建了一个叫做 `test` 的仓库，在 PicGo 里我要设定的仓库名就是 `Molunerfinn/test`。一般我们选择 `main` 分支即可。然后记得点击确定以生效，然后可以点击 `设为默认图床` 来确保上传的图床是 GitHub。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/setup_github.png)

至此配置完毕，已经可以使用了。当你上传的时候，你会发现你的仓库里也会增加新的图片了：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/success.png)
