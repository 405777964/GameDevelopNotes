Scriptable Render Pipeline Batcher  
可编程渲染管线批处理程序
可编程渲染管线 （SRP） Batcher 是一种[绘制调用优化](https://docs.unity3d.com/cn/current/Manual/optimizing-draw-calls.html)，可显著提高使用 SRP 的应用程序的性能。SRP Batcher 减少了 Unity 为使用相同着色器变体的材质准备和调度绘制调用所需的 CPU 时间。
为了准备绘制调用，CPU 会在 GPU 上设置资源并更改内部设置。这些设置统称为 render state。对==渲染状态的更改==（例如切换到其他材质）是图形 API 执行的资源最密集的操作。

## 调用顺序
可以在同一场景中==使用多个绘制调用优化方法==，但请注意，Unity 会按特定顺序确定绘制调用优化方法的优先级。如果将游戏对象标记为使用多个绘制调用优化方法，则 Unity 将使用最高优先级的方法。唯一的例外是 [SRP Batcher](https://docs.unity3d.com/cn/current/Manual/SRPBatcher.html)。使用 SRP Batcher 时，Unity 还支持对[与 SRP Batcher 兼容的游戏对象](https://docs.unity3d.com/cn/current/Manual/SRPBatcher.html#gameobject-compatibility)进行静态批处理。Unity 按以下顺序确定绘制调用优化的优先级：

1. SRP Batcher and 静态批处理
2. GPU instancing  GPU 实例化
3. Dynamic batching  动态批处理

https://docs.unity3d.com/cn/current/Manual/SRPBatcher.html