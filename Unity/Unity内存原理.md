#unity优化 
## Unity的内存
### Unity的内存起源
Unity实际可看作是一个使用C++开发的游戏引擎，它使用`.net`的脚本模拟机 （CLR）。Unity从**虚拟内存**（Virtual Memory）中给原生（C++）对象和虚拟机分配内存，同样第三方插件的内存分配也是在虚拟内存池中。

Unity在内存空间本质上可以划分为3个不同的内存域：
- 托管域：Mono平台工作的地方，编写的MonoBehaviour脚本和自定义C#类都会在此域实例化对象，内存空间自动被垃圾回收器管理。
- 本地域：Unity有一些底层的本地代码，由C++编写，根据不同平台编译到不同的应用程序中。该域关心内部内存空间的分配，如为各种子系统（渲染管线、物理系统...）分配资源数据和内存空间。也包括GameObject和component等重要游戏对象的部分本地描述，以便和这些内部系统沟通，也是大多数内建的Unity类（transform、Rigidbody等组件）保存其数据的地方。
- 外部库：如DirectX和OpenGL库，也包括项目中包含的很多自定义库和插件。在C#代码中引用这些类库将导致类似的内存上下文切换和后续成本。
托管域也包含存储在本地域中的对象描述的包装器，因此当和transform等组件交互时，大多数指令会请求Unity进入它的本地代码，在那里生成结果，接着将结果复制回托管域，这正是托管域和本地域之间本地-托管桥的由来，当两个域对相同实体有自己的描述时，跨越它们之前的桥需要内存进行上下文切换，造成相当严重的潜在性能问题。

- **原生内存**（Native Memory）是虚拟内存的一部分，它用来给所有需要的对象分配内存页面，包括**Mono堆**（`Mono Heap`）。
- **Mono堆**是出于虚拟机的需要而专门分配的本机内存的一部分。它包含了所有由`C#`分配的托管的内存类型，而这些内存的托管对象就是垃圾收集器（GC）。
Unity内部有几个专门的分配器，它们负责管理虚拟内存的短期和长期分配需求。所有的Unity资产（Assets）都是存储在原生内存中的。但这些资产会被轻量的包装成Class，以供逻辑访问和调用。*也就是说如果我们用C#创建了一张Texture，它的大部分原始数据（RawData）存在Native内存中（本地域），并且会有很小的一个Class对象进入到虚拟机中，也就是Mono堆中（托管域）。*

### Reserved
**内存分页**（IOS上大部分为16K/page）是内存管理的最小单位，当Unity需要申请内存的时候，都会以block的方式（若干个页）进行申请。如果某一页在若干次GC（IOS为8次）之后仍然为Empty，它就会被释放，实际的物理内存就会被还回给物理内存。
但是Unity是在虚拟内存中管理内存的，因此虚拟空间的地址并不会返还，所以它变成 **Reserved** 的了。这些Reserved地址空间不能再被Unity重复分配了。
如果虚拟内存地址==发生频繁分配和释放==会导致==**地址空间耗尽**==，从而被系统杀掉。
![[Pasted image 20221117003604.png]]
堆内存被GC了，这部分物理地址会空出来
![[Pasted image 20221117003824.png]]
当下一次需要申请堆内存时，**Mono堆会先检查当前堆内的空间是否存在连续的空间能容纳这次内存申请，如果不够就会进行一次GC，如果还是不够，就会执行内存扩展操作，向操作系统要更多的内存。**（Mono堆申请的物理内存是连续的，并且Mono堆向操作系统申请扩展内存**非常耗时**。所以大部分情况Mono堆尽量保持自己已经申请到的物理内存。）
![[Pasted image 20221117004023.png]]
*空出来又不能被重复利用的内存就成为了内存碎片*
*堆内存碎片会造成两个结果：游戏占用的内存越来越大，GC会更频繁地被触发*

### Profiler Simple 视图
使用Unity的Profiler进行内存分析
![[Pasted image 20221117004432.png]]
- **Used Total**: 已使用内存。
- **Unity**：所有Unity申请和管理的内存（包含Mono Heap）。
- **Mono**：托管堆内存。
- **GfxDriver**：GPU显存开销，主要由Texture，Vertex buffer，index buffer组成。
*这里的Total Reserved还不是游戏虚拟内存的精确值*
	- 它不包括游戏的二进制可执行文件。
	- GfxDriver不包括渲染目标和由驱动层分配的各种缓冲区。
	- **分析器只看到Unity代码完成的分配，看不到第三方native插件和操作系统的分配。**

### Profiler Detailed 视图
![[Pasted image 20221117005417.png]]
- **Assets**：当前从scenes，Resources和AssetBundles加载的总资源。
- **Built-in Resources**：Unity Editor资源或者Unity default资源。
- **Not Saved**：被标记为DontSave的GameObjects。
- **Scene Memory**：GameObject和它的附属的Components。
- **Other**：其他不在上面几条分类中的。

## IOS的内存管理
### IOS系统所使用的内存形态
- **Physical Memory**：IOS设备上的物理芯片内存（机器内存）。
- **Virtual Memory**：虚拟内存，也是OS给每个APP分配的虚拟地址空间。
- **GPU Driver Memory**：IOS系统使用统一架构，即GPU和CPU共享某一部分内存，如纹理和网格数据，这些是由驱动器进行分配的。另外还有一些是GPU的驱动层和GPU独享的内存类型。（Video memory）![[Pasted image 20221117010437.png]]
- **Malloc Heap**：APP实际申请内存的地方。（Unity申请内存都在这里）
- **Resident Memory**：驻留内存，这是游戏或APP实际所占用的物理内存。当应用向系统申请内存的时候，虚拟内存是直接增长的。但如果申请完的内存并没有向里面写入数据，它并不会产生实际的物理内存分配。（*图中VM分配了4个region，第3个region包括了13个page，真正映射到物理内存上的page只有6个，这6个就是Resident Memory*）![[Pasted image 20221117010716.png]]
- **Clean Memory**：Clean内存是Resident Memory的一部分。但这部分内存的类型是**只读**的。由于是只读的特性，因此它可以在应用程序内存不足的时候被Page Out。
- **Dirty Memory**：这部分无法被OS换页操作。
- **Swapped Compressed Memory**：属于Dirty内存，当应用内存不足时，OS会将Dirty内存使用频次较少的内存进行压缩存放，等需要用的时候再重新解压。
- **Footprint**：是苹果推荐的内存度量和优化的指标。当Memory Footprint的值达到Limit line会触发内存警告，导致OOM。![[Pasted image 20221117011340.png]]

### XCode视角的局限
- Unity关注的重点是虚拟内存，XCode从OS的视角更多关注物理内存。
- Unity无法统计系统库和第三方插件，XCode可以但很难区分。

## Android的内存
- RAM：内存。
- zRAM：内存不足时，OS会将RAM的一部分数据压缩存在zRAM。
- Storage：存储器，放APP、照片、缓存文件等东西。![[Pasted image 20221117011714.png]]
- **VSS-** Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）
- **RSS-** Resident Set Size 实际使用物理内存（包含共享库占用的内存）
- **PSS**- Proportional Set Size 实际使用的物理内存（比例分配共享库占用的内存）
- **USS-** Unique Set Size 进程独自占用的物理内存（不包含共享库占用的内存）