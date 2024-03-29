#Lua 
GC回收过程
	- **标记**（mark）：Lua将根集对象标记为活的，根据对象实际上就是C注册表，像主线程和全局环境都是注册表的预定义项。从根集对象一直往下标记，任何存在于活对象中的对象，只要程序可到达（弱表除外），也会被标记为活的。
	- **清理**（cleaning）：Lua遍历那些未标志的对象，如果它们有终结函数，将它们移出到另一个列表中，后面第四阶段会用到。接着Lua遍历弱表，把其中key或value未被标记的对象从弱表移除。
	- **清除**（sweep）:Lua遍历所有的对象，如果未被标记则回收掉，如果有标记则清除标记。
	- **终结**（finalization）：在清理（cleaning）阶段将有终结函数，且未标志的对象都移到一个独立的列表中，这阶段就是遍历这个列表，并调用它们的终结函数。而这些未活的对象，实际上会在下一个周期才被回收掉。

**三色增量**标记清除法（实际上是四种）
- **白色**：当前对象为待访问状态，表示对象还没被GC标记过。（如果一个对象在结束GC扫描过程后仍然是白色，则说明该对象没有被系统中的任何一个对象所引用，可以回收了。
- **灰色**：当前对象为待扫描状态，表示对象已经被GC访问过了，但是该对象引用的其他对象还没被访问到。
- **黑色**：当前对象为已扫描状态，表示对象被GC访问过了，并且该对象引用的其他对象也被访问过了。

引入灰色节点的概念后，算法不要求一次性完整执行完毕，而是可以把已经扫描但是其引用的对象还未被扫描的对象置为灰色。

[垃圾回收](https://zhuanlan.zhihu.com/p/76250195)
