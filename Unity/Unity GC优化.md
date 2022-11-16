#unity优化 
## Unity的内存起源
Unity实际可看作是一个使用C++开发的游戏引擎，它使用`.net`的脚本模拟机。Unity从**虚拟内存**（Virtual Memory）中给原生（C++）对象和虚拟机分配内存，同样第三方插件的内存分配也是在虚拟内存池中。

**原生内存**（Native Memory）是虚拟内存的一部分，它用来给所有需要的对象分配内存页面，包括Mono堆（`Mono Heap`）。

