#垃圾回收 

### Unity内存分为两种：

**原生内存** Native Memory

**托管内存** Managed Memory

Unity使用**Memory Manager**来管理内存

当Unity要malloc某个东西时，会给这个内存一个标签，就是**Memory Label，**通过Memory Label在运行时区分这块内存的归属。

### 内存分配

Unity有十几种分配器，不同分配策略对应不同的分配器

**栈分配器** Stack Allocator

**批量分配器** Batch Allocator

**动态堆分配器** Dynamic Heap Allocator

## Managed Memory

![Unity%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E5%92%8C%E5%9B%9E%E6%94%B6%E7%9A%84%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86%2089ec857861fa40378579fdffd8c2cc98/Untitled.png](Unity/内存分配与垃圾回收/Untitled.png)

蓝色代表预留的内存，绿色代表已使用内存

当已使用内存快达到预留内存时，Unity会再申请一部分内存给预留内存

当已使用内存被回收时，预留内存**不会被回收**

## Boehm GC

主流回收器有三种

**保守式回收（Conservative GC）** 以Boehm为代表

**分代式回收 （Generational GC）**以Simple Generational GC为代表

**引用（计数）式回收 （Reference Counting GC）**Java就是结合了保守式的引用式回收

Boehm回收特点：

不分代不合并内存，会产生内存碎片

所有保守式回收都是非精准式内存回收

[https://zhuanlan.zhihu.com/p/381859536](https://zhuanlan.zhihu.com/p/381859536)