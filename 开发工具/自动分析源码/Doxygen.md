Doxygen Documentation Generator，它是一个文档生成器，可以根据代码来生成文档

1. 创建配置文件，以指定生成的文档格式，可以用 `-g` 来生成默认的配置文件模板
```
doxygen -g Doxygen.config
```

2. 通过指定的配置文件运行
```
doxygen Doxygen.config
```

注意：它依赖 `Graphviz`