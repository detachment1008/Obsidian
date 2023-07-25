# 6.1 Git 对象探秘

我们使用 `git log -1 --pretty=raw` 查看历史提交记录，可以看到有三个 SHA1 哈希值表示的对象 ID
```
commit acd87429060f9c1a30dd92418bb2355268ad5af8
tree 2f02c2806fc20563fb30ae0bec77a9480fffbc4c
parent addb7b9aad1e8bce5109560a18a414d029962572
```
- `commit acd87429060f9c1a30dd92418bb2355268ad5af8`：表示本次提交唯一标识
- `tree 2f02c2806fc20563fb30ae0bec77a9480fffbc4c`：表示本次提交对应的目录树
- `parent addb7b9aad1e8bce5109560a18a414d029962572`：本次提交的父提交

使用命令：`git cat-file -t <commit-id>`，查看对应 SHA1 哈希值对应的类型
使用命令：`git cat-file -p <commit-id>`，查看对象的内容

对象都保存在 objects 目录下，并以 ID 前 2 位为目录名，后 38 位为文件名

对象的小总结：

commit 对象：五个内容，三个 SHA1 哈希值
```
// commit 对象的唯一标识
commit                     acd87429060f9c1a30dd92418bb2355268ad5af8

// 对应的 tree 对象的唯一标识
tree                       2f02c2806fc20563fb30ae0bec77a9480fffbc4c

// 上一个 commit 对象的唯一标识
parent                     addb7b9aad1e8bce5109560a18a414d029962572

// 修改者
author                     dcr <974449413@qq.com> 1690161139 +0800

// 提交者
committer                  dcr <974449413@qq.com> 1690161139 +0800
```


tree 对象：
```
// 保存着 blob 对象
100644 blob e965047ad7c57865823c7d992b1d046ea66edf78    welcome.txt
```

blob 对象：
```
文本内容
```

分支名和 HEAD：

通过一步步追踪，我们发现 HEAD 存放在 .git/HEAD ，它存放的是一个路径：refs/heads/master

而 master 就正好存放在 .git/refs/heads/master ，它的内容是一串 SHA1 值

即 master 存放的是一个 commit 对象，而 HEAD 存放的是一个路径

# 6.2 思考：SHA1 哈希值到底是什么，是如何生成的

它是将任意长度的输入经过散列算法转化为固定长度的输出，这个固定长度的输出就可以称为哈希值

# 6.3 为什么不用顺序数字来表示提交

因为 git 是分布式系统，无法保证数字具有唯一性