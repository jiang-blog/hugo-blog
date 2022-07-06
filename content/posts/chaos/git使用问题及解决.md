---
tags: []
categories: ["work"]
description:
summary:
draft: false
isCJKLanguage: true
date: "2022-06-09"
lastmod: "2022-06-09"
title: git 使用问题及解决
---

# git 使用问题及解决

记录在 git 使用过程中遇到的多种多样的问题及解决方法

## 多个 ssh 密钥

由于 github 不允许两个账号使用同一个 ssh 密钥，故尝试生成并使用两个密钥

[Windows下配置多个git账号的SSH Key - 简书 (jianshu.com)](https://www.jianshu.com/p/d195394f7d2e)

### sourcetree 管理多个 ssh 密钥

[解决SourceTree本地多ssh key的问题(Window系统)](https://blog.csdn.net/qq_26343241/article/details/103489413)

## 删除远程连接

[Git 取消远程分支关联，并关联到新的远程分支，将代码推上去_向小凯同学学习的博客-CSDN博客_git 取消关联](https://blog.csdn.net/wd2014610/article/details/79637503)

## win git 自动拉取

### 建立执行脚本

首先需要一个已建立链接的本地仓库

新建 `{name}.bat`，输入：
```bash
@echo off
D: ::(注释)进入非C盘
cd {\folder}::目标仓库文件夹
git pull
```

### 设置自动运行

#### 命令行设置

在 win 终端输入：
```bash
schtasks /create /sc monthly /mo 1 /tn "auto-gitpull" /tr "{\folder}\{name}.bat"
```
实现每月自动 git pull 一次远程仓库

- `/sc` - 指定计划类型，有效值为 MINUTE、HOURLY、DAILY、WEEKLY、MONTHLY、ONCE、ONSTART、ONLOGON、ONIDLE
- `/mo` - 在计划类型下的运行频率，对 MONTHLY 为必需
- `/tn` - 指定任务的名称
- `/tr` - 指定任务运行的程序或命令

#### 界面设置

在资源管理器的计算机或开始菜单右键单击打开计算机管理，**任务计划程序**位于**系统工具**中
或
使用快捷键 Win+R 调出运行命令，然后输入 taskschd.msc，回车。

### 其他

与 github 的 ssh 似乎需要密码不然提示无效
可通过 `ssh-keygen -p` 来删除密码（需先输入旧密码然后两次回车设置并确认新密码为空）

## github fork 后与原仓库同步

[Github进行fork后如何与原仓库同步](https://blog.csdn.net/weixin_45429089/article/details/123063747)
