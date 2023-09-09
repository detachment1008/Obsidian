# git branch
---

分支的查看，创建，删除，设置为跟踪分支

## 没有参数

默认：`git branch`：查看本地所有分支

选项：`git branch [option]`
- `-a`：查看本地所有分支+远程跟踪分支

## 一个参数

默认：`git branch <name>`：以当前分支为基底，创建一个分支

选项：`git branch [option] <branch>`
- `-d`：删除一个本地的已合并到当前分支的分支
- `-D`：删除一个本地的分支
- `-u`：设置当前分支为跟踪分支，并跟踪指定的分支

## 两个参数

默认：`git branch <name> <branch>`：以 `branch` 为基底，创建一个分支
