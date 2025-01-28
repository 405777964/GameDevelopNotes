Scriptable Render Pipeline Batcher  
可编程渲染管线批处理程序
可编程渲染管线 （SRP） Batcher 是一种[绘制调用优化](https://docs.unity3d.com/cn/current/Manual/optimizing-draw-calls.html)，可显著提高使用 SRP 的应用程序的性能。SRP Batcher 减少了 Unity 为使用相同着色器变体的材质准备和调度绘制调用所需的 CPU 时间。
为了准备绘制调用，CPU 会在 GPU 上设置资源并更改内部设置。这些设置统称为 render state。对==渲染状态的更改==（例如切换到其他材质）是图形 API 执行的资源最密集的操作。

## 调用顺序
可以在同一场景中==使用多个绘制调用优化方法==，但请注意，Unity 会按特定顺序确定绘制调用优化方法的优先级。如果将游戏对象标记为使用多个绘制调用优化方法，则 Unity 将使用最高优先级的方法。唯一的例外是 [SRP Batcher](https://docs.unity3d.com/cn/current/Manual/SRPBatcher.html)。使用 SRP Batcher 时，Unity 还支持对[与 SRP Batcher 兼容的游戏对象](https://docs.unity3d.com/cn/current/Manual/SRPBatcher.html#gameobject-compatibility)进行静态批处理。Unity 按以下顺序确定绘制调用优化的优先级：

1. SRP Batcher and 静态批处理
2. GPU instancing  GPU 实例化
3. Dynamic batching  动态批处理

## SRP Batcher 的工作原理
优化绘制调用的传统方法是减少绘制调用的数量。相反，SRP Batcher 减少了绘制调用之间的==渲染状态更改==。为此，SRP Batcher ==结合了一系列 `bind` 和 `draw` GPU 命令==。每个命令序列称为 SRP 批处理。
为了实现渲染的最佳性能，每个 SRP 批处理都应包含尽可能多的 `bind` 和 `draw` 命令。为此，请使用尽可能少的==着色器变体==。您仍然可以根据需要使用==同一着色器的任意数量的不同材质==。
![[Pasted image 20250108220142.png]]
SRP Batcher 是一个低级渲染循环，它使材质数据保留在 GPU 内存中。如果==材质内容没有更改==，则 ==SRP Batcher 不会进行任何渲染状态更改==。相反，SRP Batcher 使用==专用代码路径来更新大型 GPU 缓冲区中的 Unity 引擎属性==（Transform之类的）。
![[Pasted image 20250108220843.png]]

所以对比起动态批处理，SRP Batacher更容易实现合批。
如果要使用 [GPU 实例化](https://docs.unity3d.com/cn/current/Manual/GPUInstancing.html)，它与 SRP Batcher 不兼容。如果要使用完全相同的材质渲染许多相同的网格，则 GPU 实例化可能比 SRP Batcher 更高效。
https://docs.unity3d.com/cn/current/Manual/SRPBatcher.html