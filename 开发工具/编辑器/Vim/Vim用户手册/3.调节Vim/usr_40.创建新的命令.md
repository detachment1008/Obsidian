# 1.键映射

```
:map  普通，可视，操作符待决模式
:vmap 可视
:nmap 普通
:omap 操作符待决
:map! 插入和命令行
:imap 插入模式
:cmap 命令行模式
```

删除映射：`:unmap`

# 2.定义命令行命令

`:command DeleteFirst ldelete`

当运行 `:DeleteFirst` 时，就会使用 `:ldelete` 删除第一行
用户的命令必须大写字母开始，且不能使用X,Next,Print

# 3.自动命令

即当触发某种事件时，会被自动执行

如希望在每次写入文件时都自动替换文件尾的日期戳：
```
// 1.定义函数
:function DateInsert()
:  $delete
:  read !date
:endfunction

// 2.调用函数
// BufWritePre是自动命令触发事件，* 是匹配文件名的模式
:autocmd BufWritePre * call DateInsert()
```

命令格式：`:autocmd [group] {events} {file-pattern} [++nested] {command}`