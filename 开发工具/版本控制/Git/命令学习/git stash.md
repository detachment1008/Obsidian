# git stash
---

## 没有参数

默认：`git stash` 或 `git stash push`：贮藏当前工作目录和暂存区

应用：`git stash apply` 或 `git stash pop`：后者应用后会抛弃

抛弃：`git stash drop`

查看：`git stash list`

选项：`git stash [option]`
- `--keep-index`：不仅贮藏，还将其保留在索引中；即贮藏后，暂存区保持原样
- `-u`：贮藏时，包含未跟踪的文件
- `-a`：贮藏时，包含忽略的文件
- `-p`：交互式，可以选择文件贮藏

## 一个参数

上述的命令都可用贮藏名称来指定操作的贮藏

`git stash branch <branch>`：以指定分支名创建一个新分支，检出贮藏工作时所在的提交，重新在那应用工作，然后丢弃贮藏