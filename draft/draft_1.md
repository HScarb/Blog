title: Git 笔记
date: 2016-03-11 01:36:41
modified: 2016-03-11 01:36:41
author: Scarb
postid: $POSTID
slug: $SLUG
nicename: 
attachments: $ATTACHMENTS
posttype: post
poststatus: publish
tags: git
category: note

[TOC]
学习了[廖雪峰老师的Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)，记一下笔记。

# 1. 版本库
## 1.1 创建版本库
 - 初始化一个Git仓库，使用`git init`命令
 - 添加文件到Git仓库，分两步
  1. 使用命令`git add <file>`，
  2. 使用命令`git commit`，完成
  
## 1.2 管理版本库
 - 要随时掌握工作区的状态，使用`git status`命令。
 - 如果`git status`告诉你有文件被修改过，用`git diff`可以查看修改内容。

### 1.2.1 版本回退
 - 上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个^比较容易数不过来，所以写成`HEAD~100`。
 - HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。
 - 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
 - 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。
 
### 1.2.2 撤销修改
 1. 当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。
 2. 当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD file`，就回到了场景1，第二步按场景1操作。
 3. 已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

### 1.2.3 删除文件
 - `git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。
 - 命令`git rm`用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，**你会丢失最近一次提交后你修改的内容**。

# 2. 远程仓库