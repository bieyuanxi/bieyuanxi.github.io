+++
title = "Git 备忘录"
date = "2026-01-18"
description = "git常用命令备忘录"
draft = true

[taxonomies]
tags = ["git"]
+++

# 参考资料
- [Codeberg - Tags and Releases](https://docs.codeberg.org/git/using-tags/)
- [Semantic Versioning](http://semver.org/)

# clone
用于克隆远程仓库。由若干操作组成：创建本地空仓库、添加远程仓库地址、执行`git fetch`拉取远程数据、将远程默认分支(`main`)检出到本地工作区。

```sh
$ mkdir test
$ cd test
$ git init  # 初始化本地空仓库
$ git remote add origin https://github.com/xxx/xxx.git  # 添加远程地址
$ git fetch origin  # 拉取远程数据（成功）
$ ls  # 看不到代码文件（仅 .git 目录有元数据）
$ git checkout main  # 检出分支后，才会生成工作区文件
```

### 用例
```sh
$ git clone <repo> 
$ git clone git@github.com:bieyuanxi/bieyuanxi.github.io.git

# use <name> instead of 'origin' to track upstream
# 给上游远程仓库起一个新的名字，之后可以用 git remote -v 查看
$ git clone <repo> -o <name>
$ git clone git@github.com:bieyuanxi/bieyuanxi.github.io.git -o custom_name

$ git remote -v
custom_name git@github.com:bieyuanxi/bieyuanxi.github.io.git (fetch)
custom_name git@github.com:bieyuanxi/bieyuanxi.github.io.git (push)

# create a shallow clone of that depth
$ git clone git@github.com:bieyuanxi/bieyuanxi.github.io.git --depth <depth>
```


# fetch
更新本地已有仓库，从远程仓库拉取最新的提交记录、分支、标签等元数据到本地，但不会合并到工作区 / 当前分支，仅同步 “远程信息”。

这就是为什么远程仓库明明有提交，但是执行`git status`提示`nothing to commit, working tree clean`，因为必须执行`git fetch`之后才能同步远程仓库的改动。

```sh
# 可以使用git status 或者 git branch -vv查看当前分支追踪的远程仓库名称和分支
$ git status
Your branch is up to date with 'upstream/main'.

$ git branch -vv
* main 22b1c28 [upstream/main] This is a commit message.
```

### 用例

```sh
# 拉取当前分支关联的远程仓库所有分支、标签、提交记录等元数据，
# 同步到本地的远程追踪分支（如 origin/main、origin/dev），但不会修改本地工作区或本地分支的代码
$ git fetch
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 5 (delta 2), reused 5 (delta 2), pack-reused 0 (from 0)
Unpacking objects: 100% (5/5), 2.67 KiB | 1.34 MiB/s, done.
From github.com:bieyuanxi/bieyuanxi.github.io
   dfed93c..22b1c28  main       -> upstream/main
$ git status
On branch main
Your branch is behind 'upstream/main' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean

# 拉取指定远程仓库的所有分支
$ git fetch upstream

# 拉取指定远程仓库的指定分支
$ git fetch upstream main
```


# merge
TODO



# pull
`git pull`是 `git fetch + git merge` 的自动组合命令。它会一次性完成拉取远程数据(`git fetch`)和合并到本地分支(`git merge`)两个步骤。

### 用例
```sh
$ git pull [<repository>]

# 拉取默认远程仓库（当前本地分支关联的远程仓库）的对应默认分支（当前本地分支关联的远程追踪分支）
$ git pull

# 拉取&合并指定远程仓库的指定分支
$ git pull origin main
```



# rebase
TODO



# Tags and Releases
## 1. Tags
标签（`Tag`）是`git`的一个`feature`，是给仓库中某个特定提交（`commit`） 打的「永久性标记」，核心作用是给重要的提交（比如软件版本发布）起一个易记的名字（如 v1.0.0），方便快速定位和回溯，而非像分支那样随代码迭代变化。

### 用例

#### 1. 查看标签
```sh
$ git tag -l
```
#### 2. 创建标签
```sh
$ git tag v1.0.0          # 给当前最新提交打轻量标签
$ git tag v0.9.0 a1b2c3d  # 给指定commit打轻量标签

# -a（annotated）指定附注标签，-m 加标签说明（必填）
$ git tag -a <tagname> -m <msg>
$ git tag -a v1.0.0 -m "发布v1.0.0版本，支持XX功能"

# 给指定commit打附注标签
$ git tag <commit> -a <tagname> -m <msg>
$ git tag a1b2c3d -a v0.9.0 -m "测试版本v0.9.0"
```

#### 3. 推送标签
推送提交时标签不会自动被提交到远程仓库，需要显式操作。
```sh
# 推送单个标签
$ git push <remote target, probably "origin"> <tag name, e.g., "v1.2.3">
$ git push origin v1.0.0

# 推送本地所有未推送的标签
$ git push <remote target, probably "origin"> --tags
$ git push origin --tags  
```

#### 4. 删除标签
```sh
$ git tag -d <tagname>
```

### 所有参数
```sh
$ git tag -h
usage: git tag [-a | -s | -u <key-id>] [-f] [-m <msg> | -F <file>] [-e]
               [(--trailer <token>[(=|:)<value>])...]
               <tagname> [<commit> | <object>]
   or: git tag -d <tagname>...
   or: git tag [-n[<num>]] -l [--contains <commit>] [--no-contains <commit>]
               [--points-at <object>] [--column[=<options>] | --no-column]
               [--create-reflog] [--sort=<key>] [--format=<format>]
               [--merged <commit>] [--no-merged <commit>] [<pattern>...]
   or: git tag -v [--format=<format>] <tagname>...

    -l, --list            list tag names
    -n[<n>]               print <n> lines of each tag message
    -d, --delete          delete tags
    -v, --verify          verify tags

Tag creation options
    -a, --[no-]annotate   annotated tag, needs a message
    -m, --message <message>
                          tag message
    -F, --[no-]file <file>
                          read message from file
    --trailer <trailer>   add custom trailer(s)
    -e, --[no-]edit       force edit of tag message
    -s, --[no-]sign       annotated and GPG-signed tag
    --[no-]cleanup <mode> how to strip spaces and #comments from message
    -u, --[no-]local-user <key-id>
                          use another key to sign the tag
    -f, --[no-]force      replace the tag if exists
    --[no-]create-reflog  create a reflog

Tag listing options
    --[no-]column[=<style>]
                          show tag list in columns
    --contains <commit>   print only tags that contain the commit
    --no-contains <commit>
                          print only tags that don't contain the commit
    --merged <commit>     print only tags that are merged
    --no-merged <commit>  print only tags that are not merged
    --[no-]omit-empty     do not output a newline after empty formatted refs
    --[no-]sort <key>     field name to sort on
    --[no-]points-at <object>
                          print only tags of the object
    --[no-]format <format>
                          format to use for the output
    --[no-]color[=<when>] respect format colors
    -i, --[no-]ignore-case
                          sorting and filtering are case insensitive
```

## 2. Releases
`Release` 不是`git`的`feature`，属于`Github`或`Forgejo`之类的供应商提供的特性，允许提供打包二进制和源文件，并和`git`的`tag`联系在一起。

创建`release`参考：[Create a release on Codeberg](https://docs.codeberg.org/git/using-tags/#on-codeberg)