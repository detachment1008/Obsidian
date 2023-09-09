# git show
---

## 一个参数

默认：`git show <hash>/<branch>`：以指定的 `SHA-1` 值或分支名，查看提交对象以及引入的补丁

`git show <branch>/HEAD@{n}`：查看指定分支 `n` 次前的提交信息
`git show <branch>/HEAD@{yesterday}`：查看指定分支昨天指向了哪个提交
`git show <branch>/HEAD@{2.months.ago}`：两个月前