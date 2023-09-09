# 出生：产生Asset的方式

产生方式：
- 第三方工具导入，如：模型
- Unity中产生的，如：预制体，场景
- 脚本

另一种分类方式：
1. 运行时Asset
	- 即打成包的资源
2. 编辑期Asset
	- 它内部会额外存储一些如何打包和如何编辑的信息
	- 这些信息是不会被最终打到包体里的

# 成长：Asset导入设置

1. 重要的[[Meta文件#^be9a1e|Meta文件]]
2. 拥挤的Library文件夹
3. 神奇的StreamingAssets文件夹
	- 它可以在运行时原封不动的打入到包体内(不压缩)
	- 在安卓平台上，它可以被直接读取(因为打包安卓时，是不压缩的)
4. 害羞的波浪线

## Library文件夹

以老版本(版本1)举例：
- 关注Library文件夹下的metadata文件夹
	- 它下面有一系列以guid前两位标记的文件夹
	- 在内部有有两个对应文件
		- 无后缀
		- .info后缀

即Asset下的资源都会再导入进Library文件夹

- 如果你发现在导入新资源后，似乎没有更新：可以检查一下Library对应资源的修改日期
- 在打AssetBundle时，查看这个修改日期判断它是否应该被打入包体内

新版本(版本2)举例：它没有metadata文件夹了

它多了这四个主要的文件：
- SourceAssetDB
- SourceAssetDB-lock
- ArtifactDB
- ArtifactDB-lock

因为它使用了 Memory Data Base 来进行存储：内存数据库
- LMDB

这也是它比版本一速度更快的原因(内存比磁盘快)

## 害羞的波浪线

打开Assets文件夹，下面可能会有一个 hide~ 的文件夹：
- 所有以 ~ 结尾的文件夹都会被 Unity 忽视掉

比如我们可以用Resource文件夹日常开发处理，在打包的时候，在后面加上一个~即可让Unity忽视它，即不会打入到包体内部

# 兄弟姐妹 -- Asset与AssetBundle

AssetBundle 是Asset的一个集合

## 1.AssetBundle的原理

严格来说，AssetBundle就是一个压缩包

使用它的好处：
1. 解决资源依赖问题
2. 可以做跨平台
3. 可以做快速索引

也可以说AssetBundle是Unity的一个虚拟的文件系统

内容：
- 头：摘要信息
- 体：资源信息

Scene是一种特殊的资源，所以Asset不能和Scene打在一起

加载AssetBundle时，它的头会被立即加载进来：
- 体是按需加载的

## 2.AssetBundle的参数

1. ChunkBasedCompression：改良过的LZ4
2. DisableWriteTypeTree：帮助减小包体大小
3. DisableLoadAssetByFileName：会有CPU消耗
	- 小包体时作用不大
4. DisableLoadAssetByFileNameWithExtension：

## 3.AssetBundle的识别

## 4.AssetBundle的策略

不要走极端：
- 打的太大，对用户体验不好，下载缓慢且下载失败了还要重新下载
- 打的太小，对内存会有消耗，即头数据比例增大，造成了内存浪费

# 独当一面 -- Asset的加载及管理

1. 编辑期和运行时加载机制不同：这个编辑期指的是Unity的使用，运行时指的是发布后
	- 编辑期会都加载进来的
	- 运行时会按需加载
	- 所以不能用编辑期的Profile去衡量运行时的Profile
2. 序列化与反序列化

举一个简单的例子：A场景的三个Cube不是预制体，B场景的三个Cube是预制体
- 那么它们的场景文件大小：A > B
- 加载速度时长：A > B

- 这是因为在B场景中，它们都是通过fileid和guid指向了一个文件，相同的不会重复的去加载
- 而在A场景中，尽量这三个Cube都是一样的，但是Unity还是认为这三个是不同的东西，所以都会完整的加载进来

建议：尽量使用Prefab，不仅制作过程中比较方便，在runtime的效率也会有提升

Unity序列化也是以这个场景文件中的数据进行操作的，当数据少，序列化也就更快

3. 兼容性之树

就是 TypeTree：为了Unity跨版本之间的兼容性做的

如果保证AssetBundle和APK的版本是一致的，就不需要这个TypeTree

4. 同步与异步

同步：快，但是会造成主线程卡顿
异步：慢(至少慢一帧)，但是尽量不会造成主线程卡顿
- 如果这一帧发起，最快也要下一步再开始
- 总体时间是大于同步的

混合使用会有问题，问题如下：

5. Preload与Presistent

引擎内部，这两个负责加载的：
- Preload Manager：负责调度任务
	- 上层有任务时，会形成一个option，会交给Preload
	- 它内部有一个队列
	- 每一帧会取出一个option任务执行它
	- 如果是并行的，会在下一帧再选择一个任务去并行执行
- Presistent Manager：它的任务是把文件从硬盘上读取到内存，同时给它分配一个id
	- 在并行的过程中，option任务可能会使用Presistent Manager
	- 如果同步和异步混用，它们会去抢这个Presistent Manager，因为分配id是阻隔线程的
	- 即异步会被同步阻断或同步被异步阻断

# 驾鹤西归 -- Asset的卸载

1. 又爱又恨 -- UnloadUnusedAssets
	- 它归Preload Manager管理
2. 又爱又怕 -- AssetBundle.Unload
	- 它不归Preload Manager管理







