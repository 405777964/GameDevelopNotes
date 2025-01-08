Scriptable Render Pipeline Batcher  
可编程渲染管线批处理程序
可编程渲染管线 （SRP） Batcher 是一种[绘制调用优化](https://docs.unity3d.com/cn/current/Manual/optimizing-draw-calls.html)，可显著提高使用 SRP 的应用程序的性能。SRP Batcher 减少了 Unity 为使用相同着色器变体的材质准备和调度绘制调用所需的 CPU 时间。
为了准备绘制调用，CPU 会在 GPU 上设置资源并更改内部设置。这些设置统称为 render state。对==渲染状态的更改==（例如切换到其他材质）是图形 API 执行的资源最密集的操作。

https://docs.unity3d.com/cn/current/Manual/SRPBatcher.html