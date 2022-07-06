---
tags: []
categories: ["chaos"]
description:
summary:
draft: false
isCJKLanguage: true
date: "2022-06-09"
lastmod: "2022-06-10"
title: Hugo 建站
---

# Hugo 建站

在多次折腾博客建站，多次反复查阅教程后我意识到在未来我很可能还会再干一遍，所以记录一下，便于将来查阅

> [!cite] 参考
> [Hugo + GitHub Action，搭建你的博客自动发布系统 · Pseudoyu](https://www.pseudoyu.com/zh/2022/05/29/deploy_your_blog_using_hugo_and_github_action/)

## 准备

- 一个 github 账号
- hugo 安装 [Releases · gohugoio/hugo (github.com)](https://github.com/gohugoio/hugo/releases)
	- 解压
	- 所在目录加入环境变量
	- 验证：`hugo version`

### 初始化

`hugo new site {博客名}`

## 主题配置

本次使用 [Loveit主题](https://github.com/dillonzq/LoveIt)

### 关联主题仓库

>>我们可以将主题仓库直接 `git clone` 下来进行使用，但这种方式有一些弊端，当之后自己对主题进行修改后，可能会与原主题产生一些冲突，不方便版本管理与后续更新。我采用的是将原主题仓库 `fork` 到自己的账户，并使用 `git submodule` 方式进行仓库链接，这样后续可以对主题的修改进行单独维护。

```bash
cd {博客名}/
git init
git submodule add https://github.com/jiang-blog/LoveIt themes/LoveIt
```

设置 upstream 以便于从原仓库拉取
>[Github进行fork后如何与原仓库同步](https://blog.csdn.net/weixin_45429089/article/details/123063747)
```bash
git remote add upstream https://github.com/dillonzq/LoveIt.git
```

### 主题配置

前往 [Favicon Generator for perfect icons on all browsers (realfavicongenerator.net)](https://realfavicongenerator.net/) 获取图标

根据 [主题文档 - 基本概念 - LoveIt (hugoloveit.com)](https://hugoloveit.com/zh-cn/theme-documentation-) 中的教程配置 `config.toml` 文件

## 自动发布

### 发布操作

通过在博客文件夹目录执行命令 `hugo` 在 `/public` 文件夹生成静态网页文件，将 public 文件夹推送至 `{username}/{username}.github.io` 即可生成网页

### 自动操作

通过 Github Action 完成上面的步骤，实现每次提交更新博客源文件夹即自动构建，部署

配置文件位于仓库目录 `.github/workflow` 下，可命名为 `deploy.yml`，内容示例如下：

```yml
name: deploy

on: # Github action 触发条件
    push: # 每次推送触发
    workflow_dispatch: # 在 GitHub 项目仓库的 Action 工具栏进行手动调用

jobs: # 任务内容
    build: # 任务名
        runs-on: ubuntu-latest # 任务运行环境
        steps: # 运行步骤
            - name: Checkout
              uses: actions/checkout@v2
              with:
                  submodules: true # 获取submodule主题
                  fetch-depth: 0 # Fetch all history for .GitInfo and .Lastmod
            
            - name: Setup Hugo
              uses: peaceiris/actions-hugo@v2 # hugo官方action，用于获取hugo环境
              with:
                  hugo-version: "latest"

            - name: Build Web
              run: hugo # 构建网页

            - name: Deploy Web
              uses: peaceiris/actions-gh-pages@v3 # 自动发布github pages
              with:
                  PERSONAL_TOKEN: ${{ secrets.PERSONAL_TOKEN }} 
                  EXTERNAL_REPOSITORY: jiang849725768/jiang849725768.github.io # 发布位置
                  PUBLISH_BRANCH: master # 发布分支
                  PUBLISH_DIR: ./public # 用于发布的文件夹
                  commit_message: ${{ github.event.head_commit.message }}
```

在源仓库的 `Settings - Secrets - Actions` 中添加 `PERSONAL_TOKEN` 环境变量为 Github Pages 仓库所属用户的 Token，用于保护 Token 不被其他人看见

添加 Token 时勾选 `repo`、`workflow` 和 `admin:repo_hook`

## obsidian
