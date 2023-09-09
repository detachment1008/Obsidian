# 4.1 创建版本库并第一次提交

初始配置：
1. 查看版本：`git --version`
2. 配置姓名：`git config --global user.name "名字"`
3. 配置邮箱：`git config --global user.email 邮箱`
4. 设置别名(可选)：`git config --global alias.st status`
5. 开启颜色显示(可选)：`git config --global color.ui true`

创建仓库：
```
mkdir demo
cd demo
git init

// 如果是 1.6.5 或更新的版本
git init demo
cd demo
```

# 4.2 为什么工作区根目录下有一个 .git 目录

# 4.3 git config 命令的各参数有何区别

我们可以通过 -e 选项来进行手动配置
```
git config --global -e
```

配置文件采用 INI文件格式

## INI 文件格式

分为节与键值对：
- 节：\[节名]
- 键值对：key=value

节就相当于对键值对进行分组，后面的键值对都是它名下的。它没有显式的结束符，下一个节的开始就是它的结束，或者直接文件结束
```
;我是一个注释
[user]
         name = dcr
         email = 974449413@qq.com
[credential "https://gitee.com"]
         provider = generic
[http]
         sslVerify = false
         postBuffer = 524288000
[color]
         ui = true
```

操作：
1. 读取 value：`git config --global 节.key`
2. 更新 value：`git config --global 节.key value`

也可以用 git config 命令操作任何其他的 INI 文件，如：
```
GIT_CONFIG=test.ini git config a.b.c.d "hello world"
```

# 4.4 是谁完成了提交

如果没有用户名和邮箱，生成的 commit 号就会出错

# 4.5 是否可以随意设置提交者姓名

# 4.6 命名别名是干什么的

就是简略命令或者为了配合之前的习惯