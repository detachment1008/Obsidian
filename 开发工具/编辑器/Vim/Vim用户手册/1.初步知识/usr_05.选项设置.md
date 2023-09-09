# 1.vimrc 文件

vim 会在启动时执行这个文件里的命令

:e $MYVIMRC：快捷编辑 vimrc 文件

在 $VIMRUNTIME/defaults.vim 里放的是默认的 vimrc 文件
- 可以在自己的 vimrc 第一行加上：source $VIMRUNTIME/defaults.vim 来进行初始化

# 2.vimrc 示例解释

在 $VIMRUNTIME/vimrc_example.vim 中有示例解释

```
if v:progname =~? "evim"
  finish
endif

" 载入$VIMRUNTIME目录里的defaults.vim文件
source $VIMRUNTIME/defaults.vim

" 当覆盖一个文件时，保留一个备份，但是VMS系统除外
" 如果可用，同时也设置undofile选项，用于在一个文件中保存多层撤销信息
if has("vms")
  set nobackup
else
  set backup
  if has('persistent_undo')
    set undofile
  endif
endif

" 打开hlsearch选项，即高亮上次查找模式匹配的地方
if &t_Co > 2 || has("gui_running")
  set hlsearch
endif

" 当一行超过 78 个字符时自动换行，单仅对纯文本文件有效
" augroup vimreEx 和 argroup END 进行封装，使得 au! 可用删除自动命令"
" autocmd FileType text：定义自动命令，即当前文本为 text 时，后面的命令自动执行
" setlocal textwidth 设置 textwidth 选项为78
augroup vimrcEx
  au!
  autocmd FileType text setlocal textwidth=78
augroup END

" 如果所需的特性可用，会载入 matchit 插件
if has('syntax') && has('eval')
  packadd! matchit
endif
```

# 3.defaults.vim 文件解释

syntax on：打开文件的色彩高亮

filetype plugin indent on：启动三个非常精巧的机制
1. 文件类型探测
2. 使用文件类型相关的插件
3. 使用缩进文件

# 4.简单的键盘映射

# 5.添加软件包

软件包是一组可假如 vim 的文件。

有两种文件包：
1. 可选的
2. 启动时自动载入的

如：启用 matchit 插件
```
packadd! matchit
```

# 6.添加插件

插件是一个在 vim 启动时能够被自动执行的脚本

插件目录：~/.vim/plugin/

放在这个目录下的插件会被自动执行

两种插件：
1. 全局插件
2. 文件类型插件

# 7.添加帮助

如果插件有帮助文件，可用进一步安装帮助文件

1. 将插件放入到 plugin 文件夹中
2. 将帮助文档放入到 doc 文件夹中
3. 使用 :helptags ~/.vim/doc
4. 然后就可用使用这个帮助文档了

# 8.选项窗口

:options

# 9.常用选项