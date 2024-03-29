# 1.编辑另一个文件

`:edit foo.txt`：关闭当前文件，并打开另一个
- 如果当前文件被修改且没有存储，就会报错
- 如要放弃修改，可用在命名后使用 ! 来强制操作
- 如果不想关闭当前文件 `hide edit foo.txt`

# 2.文件列表

vim 在启动时，可用打开一堆文件：`vim one.c two.c three.c`

编辑第二个：`:next`

放弃当前修改，编辑下一个文件：`:next!`

报存并编辑下一个：`:wnext`

## 我在哪儿

1. 注意标题：`2 of 3`
2. 查看完整文件列表：`:args`
	- 列优先
	- \[] 表示当前文件
3. 移动到前一个：`:previous`
4. 同理，保存移动：`:wprevious`
5. 最后一个：`:last`
6. 第一个：`:first`

自动保存：`set autowrite`

编辑另一个文件列表：`:args five.c six.c seven.h`

# 3.从一个文件跳转到另一个文件

CTRL^：跳回到上一个文件中

或者使用 \` 命令

m + 小写标记是单个文件中专用的，但是大写标记是任何文件都可用使用的

查看标记：`:mark M`
多个参数：`:mark MCP`

# 4.备份文件

生成备份文件：`set backup`
修改备份文件扩展名：`set backupext=.bak`

保留原始文件：`set patchmode.orig`

# 5.文件间拷贝文本

寄存器会记住拷贝的是一整行还是一个列块

# 6.显示文件

只读模式启动 vim：`vim -R file`
或者：`view file`

此时，还是可用修改文件的，因为可能你只是想排列这些文本，方便阅读。它只是禁止保存

如果不希望修改文件：`vim -M file`

# 7.修改文件名

保存：
```
:saveas <file>
```

只希望修改，但是不想立即保存：
```
:file <file>
```

源文件也没有被覆盖(似乎)