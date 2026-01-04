---
title: Git_Use
date: 2025-2-28 18:00:00
toc_number: true
tags: 
  - Git
categories: 
  - Memo
description: Git使用注意事项
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202502220802402.jpg
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202502220802222.jpg
abbrlink: 7
---

## Git在不同场景下使用的注意事项

### 场景一

#### 本地项目已基本完成，推送至远程仓库与队友共享

- 初始化仓库

```bash
git init
```

- 添加远程仓库

```bash
git remote add <remote_repository_name> <remote_repository_URL>
```

- 添加文件到暂存区

```bash
git add .
```

- 提交更改

```bash
git commit -m "Initial commit"
```

- 创建本地分支

```bash
git checkout -b new-branch-name
-b: 创建并切换到一个新分支
```

- 重命名本地分支

```bash
git branch -m master main
```

- 推送到远程仓库

```bash
git push -u origin main
-u: 指定远程仓库中的分支作为本地分支的上游分支
-f: 强制推送
```

### 场景二

#### Git换行符

`core.autocrlf` 是 Git 中用于控制换行符自动转换的配置选项

会出现如下报错信息，下次接触 —— commit时自动修改，无需过于关注：

```bash
warning: in the working copy of '.idea/.gitignore', CRLF will be replaced by LF the next time Git touches it
```

- true

**作用**：在 `checkin` 时将 CRLF 转换为 LF，在 `checkout` 时将 LF 转换为 CRLF。
适用场景：适用于 Windows 用户，希望在本地文件系统中使用 CRLF 换行符，但在仓库中存储 LF 换行符。

```bash
git config --global core.autocrlf true
```

- input

作用：在 `checkin` 时将 CRLF 转换为 LF，但在 `checkout` 时不进行转换，保持 LF 不变。
适用场景：适用于跨平台开发，希望在仓库中始终使用 LF 换行符，但在 Windows 上提交代码时自动转换为 LF。

```bash
git config --global core.autocrlf input
```

- false

作用：不进行任何换行符的自动转换。
适用场景：适用于不希望 Git 自动处理换行符的情况，通常用于特定项目或文件类型。

```bash
git config --global core.autocrlf false
```