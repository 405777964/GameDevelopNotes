#unity #热更新

[[转载]Unity用户手册-AssetBundle - 技术专栏 - Unity官方开发者社区](https://developer.unity.cn/projects/zhuan-zai-unityyong-hu-shou-ce-assetbundle)

AssetBundle 具有如下优点：

- 可以按需加载和释放 Asset 。
- 自带压缩算法，可以按需定制，减少包体容量。
- 自带版本更新内容，可以进行增量更新。
- 内部包含了对 Object 的引用关系。


https://zhuanlan.zhihu.com/p/1922333689280468099

TypeTree
告诉Unity这个ab是什么类型的资源，当升级Unity后用旧版本的资源包，也能正确解析。
如果Unity检测到这些资源的格式不匹配（资产改变了格式，在两个），