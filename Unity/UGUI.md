### Canvas
子画布只是嵌套在另一个画布组件中的画布组件。子画布将他们的孩子与父母隔离开来;脏子不会强制父级重建其几何图形，反之亦然。在某些情况下，情况并非如此，例如，对父 Canvas 的更改会导致子 Canvas 调整大小。（动静分离）

渲染细节

在 Unity UI 中编写用户界面时，请记住，Canvas 绘制的所有几何图形都将在“透明”队列中绘制。也就是说，Unity UI 生成的几何图形将始终使用 Alpha 混合从后到前绘制。从性能角度来看，要记住的重要一点是，将从多边形栅格化的每个像素进行采样，即使它完全被其他不透明多边形覆盖。在移动设备上，这种高水平的过度绘制可能会迅速超过 GPU 的填充率容量。

分析探查器结果

收集分析数据后，可能会得出几个结论。如果 Canvas.BuildBatch 或 Canvas：：UpdateBatches 似乎使用了过多的 CPU 时间，则可能的问题是单个 Canvas 上的 Canvas Renderer 组件过多。请参阅画布步骤的拆分画布部分。

如果在 GPU 上绘制 UI 所花费的时间过多，并且帧调试器指示片段着色器管道是瓶颈，则 UI 可能超出了 GPU 能够达到的像素填充率。最可能的原因是 UI 过度绘制。请参阅填充率、画布和输入步骤的修正填充率问题部分。

如果图形重建使用了过多的 CPU，如很大一部分 CPU 时间转到 Canvas.SendWillRenderCanvases 或 Canvas：：SendWillRenderCanvases，则需要进行更深入的分析。图形重建过程的某些部分可能是负责任的。

如果 WillRenderCanvas 的很大一部分是在IndexedSet_Sort或CanvasUpdateRegistry_SortLayoutList中花费的，那么就会花费时间对脏布局组件列表进行排序。考虑减少画布上布局组件的数量。有关可能的补救措施，请参阅将布局替换为“矩形转换”和“拆分画布”部分。

如果似乎在Text_OnPopulateMesh上花费了过多的时间

，那么罪魁祸首就是文本网格的生成。有关可能的补救措施，请参阅“最佳拟合”和“禁用画布”部分，如果正在重建的大部分文本实际上并未更改其基础字符串数据，请考虑拆分画布中的建议。

如果时间花在Shadow_ModifyMesh或Outline_ModifyMesh（或 ModifyMesh 的任何其他实现）上，那么问题在于计算网格修饰符所花费的时间过多。考虑删除这些组件并通过静态图像实现其视觉效果。

如果 Canvas.SendWillRenderCanvases 中没有特定的热点，或者它似乎在每一帧中都运行，则问题可能是动态元素已与静态元素组合在一起，并迫使整个 Canvas 过于频繁地重新生成。请参阅拆分画布步骤。

文本网格专业文本

TextMesh Pro（TMP）是Unity现有文本组件（如Text Mesh和UI Text）的替代品。TextMesh Pro 使用签名距离字段 （SDF） 作为其主文本呈现管线，从而可以在任何点大小和分辨率下干净利落地呈现文本。TextMesh Pro 使用一组旨在利用 SDF 文本渲染功能的自定义着色器，只需更改材质属性即可添加视觉样式（如膨胀、轮廓、柔和阴影、斜角、纹理、发光等），并通过创建/使用材质预设来保存和调用这些视觉样式，从而动态更改文本的视觉外观。

禁用画布

显示或隐藏 UI 的离散部分时，通常会在 UI 的根目录中启用或禁用游戏对象。这可确保禁用的 UI 中没有组件接收输入或 Unity 回调。

但是，这也会导致画布丢弃其 VBO 数据。重新启用画布将需要画布（以及任何子画布）运行重新生成和重新批处理过程。如果这种情况经常发生，则 CPU 使用率增加可能会导致应用程序的帧速率断断续续。

一种可能但黑客入侵的解决方法是将要显示/隐藏的 UI 放在其自己的 Canvas 或 Sub-canvas 上，然后仅在此对象上启用/禁用 Canvas 组件。这将导致 UI 的网格无法绘制，但它们将保留在内存中，并且将保留其原始批处理。此外，不会 在 UI 的层次结构中调用

[OnEnable](http://docs.unity3d.com/ScriptReference/MonoBehaviour.OnEnable.html) 或

[OnDisable](http://docs.unity3d.com/ScriptReference/MonoBehaviour.OnDisable.html) 回调。

但请注意，这不会禁用隐藏 UI 中的任何 MonoBehavior，因此这些 MonoBehavior 仍将收到 Unity 生命周期回调，例如 Update。

关键字

动静分离。

填充率导致Overdrall消耗。

Canvas绘制UI是从后往前绘制。

隐藏UI会导致Canvas丢失其VBO，重新启用会重新生成和重新批处理。

将UI挪到摄像机看不见的地方，并加上NoCamera的层级。