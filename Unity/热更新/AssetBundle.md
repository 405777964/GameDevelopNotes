#unity #热更新

[[转载]Unity用户手册-AssetBundle - 技术专栏 - Unity官方开发者社区](https://developer.unity.cn/projects/zhuan-zai-unityyong-hu-shou-ce-assetbundle)

AssetBundle 具有如下优点：

- 可以按需加载和释放 Asset 。
- 自带压缩算法，可以按需定制，减少包体容量。
- 自带版本更新内容，可以进行增量更新。
- 内部包含了对 Object 的引用关系。

官方建议使用LZ4


https://zhuanlan.zhihu.com/p/1922333689280468099

TypeTree
告诉Unity这个ab是什么类型的资源，当升级Unity后用旧版本的资源包，也能正确解析。
如果Unity检测到这些资源的格式不匹配（资产改变了格式，在两个Unity版本间），Unity可以使用TypeTree一个叫做安全二进制读取(Safe Binary Read)的东西，它将查看这个类型树，并且无论如何都要尝试构造这个对象，但是存在性能成本，比流式加载更慢。
资产类型越多，类型树越大，可以打AB的时候禁用它，让AB更小，让运行时内存更小。
建议（通常保持开启会更好）

FileRead + AssetBundle.CreateFromMemory + AssetBundle.Load
通过文件操作加载资源文件，再通过AssetBundle.CreateFromMemory的方式将字节数据转成AssetBundle格式，最后通过AssetBundle.Load加载某个资源
消耗了2倍的内存及1.3倍左右的算力，但是可以加入自定义功能，比如加解密操作

AssetBundle.CreateFromFile + AssetBundle.Load
通过直接加载文件变成AssetBundle的方式，再通过AssetBundle.Load获得资源
好处是 不会把整个资源文件都加载到内存中，而是会先加载文件中的数据头，通过数据头中的数据识别各资源在文件中的偏移位置。对应偏移量加载对应的资源位置加载到内存（按需加载）