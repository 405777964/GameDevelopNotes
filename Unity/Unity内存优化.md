#unity优化 
Unity内部由两个内存管理池：
- 堆内存（heap）主要用来存储较大的和存储时间较长的数据。
- 堆栈内存（stack）主要用来存储较小的和短暂的数据。

### 堆栈内存分配和回收
堆栈上的内存分配和回收十分快捷简单，因为堆栈只会存储短暂的或者较小的变量。内存分配和回收都以一种顺序和大小可控制的形式进行。

### 堆内存分配和回收
堆内存上的内存分配和存储相对而言更加复杂，其内存分配和回收顺序不可控。
![[Unity内存原理#^6ec959]]
### 何时会出发垃圾回收
- 堆内存上进行内存分配操作而内存不足的时候
- GC自动触发
- GC被强制执行
*意味着频繁在堆内存上进行内存分配和回收会触发频繁的GC操作*

## 降低GC的影响的方法
1. 减少GC的运行次数
2. 减少单次GC的运行时间
3. 将GC的运行时间延迟，避免在关键时候触发，如可在场景加载的时候调用GC

策略：
1. 重构游戏，减少堆内存的分配和引用的分配。更少的变量和引用会==减少GC操作中的检测个数==从而提高GC运行效率。
2. 降低堆内存分配和回收的频率（更少的事件触发GC操作）。

### 减少内存垃圾的数量
- **缓存**：在代码中反复调用操作堆内存分配的函数，我们可以缓存这些变量重复利用。（如GetComponent和每次操作都new新的数组）
- **不要在频繁调用的函数（如Update）里反复进行堆内存分配**
- **清空数据重复使用堆内存分配的对象**（对象池）
- **字符串连接**
- **Unity函数调用**：在Unity中如果函数需要返回一个数组，则一个新的数组会被分配出来用作结果返回，这不容易被注意到，特别是如果该函数含有迭代器，下面的代码中对于每个迭代器都会产生一个新的数组：
```C#
void ExampleFunction()
{
    for(int i=0; i < myMesh.normals.Length;i++)
    {
        Vector3 normal = myMesh.normals[i];
    }
}
//**对于这样的问题，我们可以缓存一个数组的引用，这样只需要分配一个数组就可以实现相同的功能，从而减少内存垃圾的产生：**//
void ExampleFunction()
{
    Vector3[] meshNormals = myMesh.normals;
    for(int i=0; i < meshNormals.Length;i++)
    {
        Vector3 normal = meshNormals[i];
    }
}
```
*此外另外的一个函数调用GameObject.name 或者 GameObject.tag也会造成预想不到的堆内存分配，这两个函数都会将结果存为==新的字符串返回==，这就会造成不必要的内存垃圾，对结果进行缓存是一种有效的办法，但是在Unity中都对应的有相关的函数来替代。对于比较gameObject的tag，可以采用GameObject.CompareTag()来替代。*
- **装箱操作**
- **协程**：调用 StartCoroutine()会产生少量的内存垃圾，因为unity会**生成实体**来管理协程。所以在游戏的关键时刻应该限制该函数的调用。基于此，任何在游戏关键时刻调用的协程都需要特别的注意，特别是包含延迟回调的协程。
- **函数引用**：函数的引用，无论是指向匿名函数还是显式函数，在unity中都是**引用类型变量**，这都会在堆内存上进行分配。匿名函数的调用完成后都会增加内存的使用和堆内存的分配。具体函数的引用和终止都取决于操作平台和编译器设置，但是如果想减少GC最好减少函数的引用。
- **LINQ和常量表达式**：由于LINQ和常量表达式以装箱的方式实现，所以在使用的时候最好进行性能测试。

## 重构代码来减小GC的影响
即使我们减小了代码在堆内存上的分配操作，代码也会增加GC的工作量。最常见的==增加GC工作量的方式是让其检查它不必检查的对象==。struct是值类型的变量，==但是如果struct中包含有引用类型的变量，那么GC就必须检测整个struct==。如果这样的操作很多，那么GC的工作量就大大增加。在下面的例子中struct包含一个string，那么整个struct都必须在GC中被检查：
```C#
public struct ItemData
{
    public string name;
    public int cost;
    public Vector3 position;
}
private ItemData[] itemData;

//我们可以将该struct拆分为多个数组的形式，从而减小GC的工作量：
private string[] itemNames;
private int[] itemCosts;
private Vector3[] itemPositions;
```
另外一种在代码中增加GC工作量的方式是保存不必要的Object引用，在进行GC操作的时候会对堆内存上的object引用进行检查，越少的引用就意味着越少的检查工作量。在下面的例子中，当前的对话框中包含一个对下一个对话框引用，这就使得GC的时候会去检查下一个对象框：
```C#
public class DialogData
{
     private DialogData nextDialog;
     public DialogData GetNextDialog()
     {
           return nextDialog;
     }
}

//通过重构代码，我们可以返回下一个对话框实体的标记，而不是对话框实体本身，这样就没有多余的object引用，从而减少GC的工作量：
public class DialogData
{
     private int nextDialogID;
     public int GetNextDialogID()
     {
           return nextDialogID;
     }
}
```
## 定期执行GC操作
`System.GC.Collect()`
通过主动的调用，我们可以主动驱使GC操作来回收堆内存。

https://www.cnblogs.com/zblade/p/6445578.html