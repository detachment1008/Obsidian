# 10.1 底层命令与上层命令

当我们创建一个 git 目录时，会有以下的内容：七个文件或目录
```
HEAD
config            // 暂不关注
description       // 暂不关注
hooks/            // 暂不关注
info/             // 暂不关注
objects/
refs/
```
- config：包含项目特有的配置选项
- description：仅供 GitWeb 程序使用
- hooks 目录：包含客户端或服务端的钩子脚本
- info 目录：包含一个全局性排除文件，用以放置那些不希望被记录在 .gitignore 文件中的忽略模式

# 10.2 Git 对象

Git 是一个内存寻址文件系统，即核心是一个键值对数据库。即可以向 Git 仓库中插入任意类型的内容，它会返回一个唯一的键。通过该键可以在任意时刻再次取回该内容。

## object 目录

首先查看该目录：
```
objects/pack
objects/info
```
可以发现，Git 默认为我们创建了两个子目录 pack 和 info，它们目前都是空的。

我们可以通过命令 hash-object 创建一个新的数据对象，并将它手动存入新的 Git 数据库中：
```
bole@dcrdeiMac demo % echo "test content" | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```
- -w ：指示该命令不要只返回键，还要将该对象写入数据库中
- --stdin ：指示该命令从标准输入中读取内容；如不指定此选项，需要在命令尾部给出待存储的文件的路径

可以发现，它返回了一个长度为 40 的字符串：d670460b4b4aece5915caf5c68d12f560a9fe3e4
- 这是一个 SHA-1 哈希值
- 一个待存储的数据外加一个头部信息(header)一起做 SHA-1 校验运算而得到的校验和

此时，我们再次查看 object 目录：
```
bole@dcrdeiMac demo % find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```
- -type f：将目录以及子目录中的所有文件列出

可以发现，此时多了一个目录：d6，并且多了一个文件：70460b4b4aece5915caf5c68d12f560a9fe3e4。
不难发现，目录就是 SHA-1 哈希值的前 2 位，文件名就是 SHA-1 哈希值的后 38 位

此时，我们直接查看这个文件的内容：
```
cat .git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
xK��OR04f(I-.QH��+I�+�K�
```
发现它输出了一段乱码，这是因为该键值对数据库在存储的时候，将文件内容进行了压缩

引入一个关键命令：`git cat-file`
- `-t`：查看类型
- `-p`：查看内容
- 注意：它的参数不再是文件名，而是键

使用 `cat-file` 命令查看刚刚文件的内容：
```
bole@dcrdeiMac demo % git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```
发现，它存储的就是刚刚完整的输入内容

此时，我们就可以用这个方式进行数据库存储了：
```
bole@dcrdeiMac demo % echo "version 1" > test.txt
bole@dcrdeiMac demo % git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30

bole@dcrdeiMac demo % find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
```

但是此时，我们更新 test.txt 文件中的数据，并再次存储试试：
```
bole@dcrdeiMac demo % echo "version 2" > test.txt
bole@dcrdeiMac demo % git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

bole@dcrdeiMac demo % find .git/objects -type f
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30

bole@dcrdeiMac demo % git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30
version 1

bole@dcrdeiMac demo % git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
version 2
```
可以发现，对象数据库记录下了该文件的两个不同版本，它不仅包含了第二条内容，还包含了第一条内容

我们可以删除掉 text.txt 文件，并从键值对数据库中取回它的任意一个版本：
```
bole@dcrdeiMac demo % rm test.txt
// 第一个版本
bole@dcrdeiMac demo % git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
bole@dcrdeiMac demo % cat test.txt
version 1
// 第二个版本
bole@dcrdeiMac demo % git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a > test.txt
bole@dcrdeiMac demo % cat test.txt
version 2
```

这其实已经完成了一个文件的存储，恢复，版本更替等功能了。这样的数据库中的一个对象，它被称之为 blob 对象
```
bole@dcrdeiMac demo % git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
blob
```

但是它有两个问题：
1. 一个文件的 SHA-1 值很长，记住它不现实
2. 它只保存了文件的内容，没有保存文件名和它的路径

所以我们需要解决这两个问题

## 树对象

可以想象为一颗树状的数据结构，子节点是 blob 对象，中间节点就是 tree 对象。所以一个 tree 对象可能包含了很多 tree 对象，也可能包含了很多 blob 对象，它们可以同时存在

查看某项目的最新树对象，可能是这样的：
```
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib
```
如：我刚刚提交了的 test.txt
```
bole@dcrdeiMac demo % git cat-file -p "master^{tree}"
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	test.txt
```
即它内部先存储了一个文件模式，然后存储了对象类型，然后就是它的键了，最后是文件名
- 100644：表示普通文件。第一个1表示这是一个普通文件，后面的00644则表示文件的权限设置为644，即所有者具有读写权限，其他人只有读权限。
- 040000：表示子目录。第一个0表示这是一个特殊类型的文件（即目录），后面的40000则表示目录的权限设置为755，即所有者和其他人都具有读取、执行和浏览权限，但只有所有者可以写入。

所以树对象是最重要的，我们通过它就可以获取当前整个目录中的所有文件以及文件名以及文件结构。

我们也可以轻松创建一个树对象。通常，Git 是根据暂存区中表示的状态来创建并记录一个树对象的，即树对象由暂存区生成。

我们可以通过底层命令 `git update-index` 为一个单独的文件创建一个暂存区
```
bole@dcrdeiMac demo % git update-index --add --cacheinfo 100644 \
  83baae61804e65cc73a7201a7252750c76066a30 test.txt
```
- --add：因为此前该文件不在暂存区中
- --cacheinfo：因为要添加的 blob 对象位于 Git 数据库中，而不是当前目录下的
- 需要指定文件模式，SHA-1 和文件名

我们使用 `git write-tree` 命令，可以将暂存区写入一个树对象
```
bole@dcrdeiMac demo % git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579
```

然后查看这个对象：
```
bole@dcrdeiMac demo % git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
100644 blob 83baae61804e65cc73a7201a7252750c76066a30	test.txt

bole@dcrdeiMac demo % git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579
tree
```
可以发现，它确实是一个树对象，只是内部只存储了暂存区中的这个 blob 对象

我们可以创建一个大一点的树对象：

先创建暂存区：
```
bole@dcrdeiMac demo % clear
bole@dcrdeiMac demo % echo "new file" > new.txt
bole@dcrdeiMac demo % git update-index --add --cacheinfo 100644 \
  1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
bole@dcrdeiMac demo % git update-index --add new.txt
```
此时，暂存区中包含了两个文件，一个是 test.txt，一个是 new.txt

然后创建树对象：
```
bole@dcrdeiMac demo % git write-tree
0155eb4229851634a0f03eb265b69f5a2d56f341
```

查看这个树对象的内容：
```
bole@dcrdeiMac demo % git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	test.txt
```

我们可以把第二个树对象再加入到自己，即使其自己成为自己的子目录，通过调用 `git read-tree` 命令
```
bole@dcrdeiMac demo % git read-tree --prefix=bak 0155eb4229851634a0f03eb265b69f5a2d56f341

bole@dcrdeiMac demo % git write-tree
dd325c8d87b9087ddcf69e9455743f7e2fce6c8e

bole@dcrdeiMac demo % git cat-file -p dd325c8d87b9087ddcf69e9455743f7e2fce6c8e
040000 tree 0155eb4229851634a0f03eb265b69f5a2d56f341	bak
100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	test.txt
```
- --prefix ：将一个已有的树对象作为子树读入暂存区

此时还有一个问题，此时虽然树对象存储了整个目录的结构，文件名，目录名，文件 SHA-1 值等信息，但是我们好像还是得记住一个树对象的 SHA-1 值，而且在创建树对象时，还是需要所有的 SHA-1 值

提交对象会解决这个问题

## 提交对象

树对象保存了整个目录的结构和对应的文件，但是我们不知道谁保存了这个树对象，也不知道它是什么时刻保存的，即当我们需要用到其中一个时，我们也不知道使用哪个树对象了

提交对象能解决这个问题，它能保存刚刚说的这些信息。

可以调用 `commit-tree` 命令创建一个提交对象，它需要一个树对象的 SHA-1 值和它的父树对象的 SHA-1 值(如果有的话)：
```
bole@dcrdeiMac demo % echo 'first commit1' | git commit-tree d8329f
a79389a904ea19c215ce04239c14ddec762949c7
```

然后查看这个新提交的对象：
```
bole@dcrdeiMac demo % git cat-file -p a793
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author dengchaoran <dengchaoran@bolegames.com> 1690257831 +0800
committer dengchaoran <dengchaoran@bolegames.com> 1690257831 +0800

first commit1
```
- 它先指定了一个顶层树对象：代表当前项目的快照
- 然后是可能存在的父提交：刚刚的提交对象不存在任何父提交
- 然后是作者和提交者信息：信息就是名字和邮箱，然后外加一个时间戳
- 留空一行
- 提交的注释

我们可以再创建两个提交对象：
```
bole@dcrdeiMac demo % echo 'second commit' | git commit-tree 0155eb -p a793
168c6682e0b82702b8a052e84b35e274034f44b5

bole@dcrdeiMac demo % echo 'third commit'  | git commit-tree dd325c -p 168c
671bbbd7bb30f88b862cf56baf0b677cf7f64958
```

然后在最后一个提交的 SHA-1 值运行 `git log` 命令：
```
bole@dcrdeiMac demo % git log --stat 671b
commit 671bbbd7bb30f88b862cf56baf0b677cf7f64958
Author: dengchaoran <dengchaoran@bolegames.com>
Date:   Tue Jul 25 12:09:55 2023 +0800

    third commit

 bak/new.txt  | 1 +
 bak/test.txt | 1 +
 2 files changed, 2 insertions(+)

commit 168c6682e0b82702b8a052e84b35e274034f44b5
Author: dengchaoran <dengchaoran@bolegames.com>
Date:   Tue Jul 25 12:08:13 2023 +0800

    second commit

 new.txt  | 1 +
 test.txt | 2 +-
 2 files changed, 2 insertions(+), 1 deletion(-)

commit a79389a904ea19c215ce04239c14ddec762949c7
Author: dengchaoran <dengchaoran@bolegames.com>
Date:   Tue Jul 25 12:03:51 2023 +0800

    first commit1

 test.txt | 1 +
 1 file changed, 1 insertion(+)
```

此时，我们没有借助任何上层命令，通过几个简单的底层操作就完成了一个提交历史的创建。即 `git add` 和 `git commit` 命令就是将被改写的文件保存为数据对象，然后更新暂存区，生成树对象，最后创建一个提交对象，指明了顶层的树对象和父提交对象

提交对象，树对象，数据对象：均以单独文件的形式保存在 .git/objects 目录下

## 是如何生成 SHA-1 哈希值的？

通过 Ruby 脚本和 SHA-1 digest 库：
```
bole@dcrdeiMac demo % irb

WARNING: This version of ruby is included in macOS for compatibility with legacy software.
In future versions of macOS the ruby runtime will not be available by
default, and may require you to install an additional package.

irb(main):001:0> content = "what is up, doc?"
=> "what is up, doc?"
irb(main):002:0> header = "blob #{content.length}\0"
=> "blob 16\u0000"
irb(main):003:0> store = header + content
=> "blob 16\u0000what is up, doc?"
irb(main):004:0> require 'digest/sha1'
=> true
irb(main):005:0> sha1 = Digest::SHA1.hexdigest(store)
=> "bd9dbf5aae1a3862dd1526723246b20206e5fc37"
```

然后再用 Git 生成一下：
```
bole@dcrdeiMac demo % echo -n "what is up, doc?" | git hash-object --stdin
bd9dbf5aae1a3862dd1526723246b20206e5fc37
```

可以发现，它们完全一样

后续 Git 会通过 zlib 压缩这条内容，然后确定路径，并写入磁盘，当子目录不存在时，就进行创建，然后通过 File.open() 打开这个文件，通过 write() 函数写入刚刚压缩过的内容，这样就大功告成了

## 总结

```
// 查看对象类型或内容
git cat-file [-t] [-p]


// 创建 blob 对象
git hash-object [-w] [-stdin]

// 创建暂存区并添加 blob 对象
git update-index [--add] [--cacheinfo]
// 向暂存区添加树对象
git read-tree [--prefix=bak]
// 暂存区写入树对象
git write-tree

// 创建提交对象，并指定父对象
git commit-tree <SHA-1> [-p] <SHA-1>
```

# 10.3 Git 引用

因为 SHA-1 值太难记了，即使我们用 commit 对象保存着 tree 对象，然后用 tree 对象保存着 blob 对象，但是似乎我们还是需要记一个 SHA-1 值，即这个 commit 对象的 SHA-1 的值

所以我们需要一个引用，即用一个别名来记住这个 SHA-1 值

## refs 目录

我们查看 refs 目录
```
bole@dcrdeiMac refs % tree
.
├── heads
│   └── master
└── tags
```
可以发现，它的内部有两个子目录 heads 和 tags，而且 heads 内有一个 master 文件

我们查看 master 文件的内容：
```
bole@dcrdeiMac refs % cat heads/master
a43ebcde4bf1611ac5542b3cb446574fd5dcfd49
```
可以发现，它保存了一个 SHA-1 值

即我们通过记住这个简单的 master 文件名，就可以找到一个 commit 对象的 SHA-1 值了

可以直接改这个文件从而让它指向另外的 commit 对象，不过不提倡，有一个更加安全的命令：
```
git update-ref refs/heads/master 671bbbd7bb30f88b862cf56baf0b677cf7f64958
```

改变成功：
```
bole@dcrdeiMac refs % git log master
commit a43ebcde4bf1611ac5542b3cb446574fd5dcfd49 (HEAD -> master)
Author: dengchaoran <dengchaoran@bolegames.com>
Date:   Tue Jul 25 11:26:39 2023 +0800

    first

bole@dcrdeiMac refs % git update-ref refs/heads/master 671bbbd7bb30f88b862cf56baf0b677cf7f64958

bole@dcrdeiMac refs % git log master
commit 671bbbd7bb30f88b862cf56baf0b677cf7f64958 (HEAD -> master)
Author: dengchaoran <dengchaoran@bolegames.com>
Date:   Tue Jul 25 12:09:55 2023 +0800

    third commit

commit 168c6682e0b82702b8a052e84b35e274034f44b5
Author: dengchaoran <dengchaoran@bolegames.com>
Date:   Tue Jul 25 12:08:13 2023 +0800

    second commit

commit a79389a904ea19c215ce04239c14ddec762949c7
Author: dengchaoran <dengchaoran@bolegames.com>
Date:   Tue Jul 25 12:03:51 2023 +0800

    first commit1
```

当我们运行类似 `git branch <branch>` 这样的命令时，Git 实际上就会运行 `update-ref` 命令，取得的当前分支最新提交对应的 SHA-1 值，并将其加入到任何新引用中
- 引用就是分支，分支就是一个文件名，包含了一个 SHA-1 值

但是还有一个问题，如何知道最新提交的 SHA-1 值呢

## HEAD 引用

HEAD 文件通常是一个符号引用，它指向目前所在的分支：
- 符号引用：表示它指向其他引用的指针

查看 HEAD 文件：
```
bole@dcrdeiMac .git % cat HEAD
ref: refs/heads/master
```

当我们切换分支时，其实就是更改 HEAD 文件的内容

当我们执行 `git commit` 时，该命令会创建一个提交对象，并用 HEAD 文件中的那个引用指向的 SHA-1 值去设置其父提交字段

同样可以手动编辑该文件，但是有更安全的命令：
```
// 查看
git symbolic-ref HEAD

// 设置
git symbolic-ref HEAD refs/heads/test
```

## 标签引用

它是第四种对象，标签对象。它非常类似于一个提交对象：
- 创建者信息
- 日期
- 注释
- 一个指针：但是它不是指向树对象，而是指向一个提交对象

它像是一个永不移动的分支引用，永远指向同一个提交对象

它放在 refs/tags 文件夹

```
// 创建轻量标签对象
git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d

// 创建一个附注对象：它会创建一个标签对象，并记录一个引用来指向该标签对象，而不是直接指向提交对象
git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9 -m 'test tag'
```

## 远程引用

它存储在 refs/remotes 文件夹

```
bole@dcrdeiMac .git % tree refs/remotes/
refs/remotes/
└── origin
    ├── @bcy_0620_theme10155
    ├── @ccz_0613_theme10117
    ├── @lw_0907_theme10141
    ├── @unicorn_0509_localization
    ├── HEAD
    ├── PokerGame
    ├── dev
    └── master
```

我们可以添加一个远程库：
```
git remote add origin git@github.com:schacon/simplegit-progit.git
```

而远程库中的文件，就是对应的分支，内部存储的值就是本地最近一次与服务器通信时对应的 SHA-1 值

它的只读的，虽然可以通过 git checkout 切换到远程引用，但是不会将 HEAD 引用指向该远程引用