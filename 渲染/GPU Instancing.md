#unity优化 
使用 GPU 实例化可使用少量[绘制调用](http://docs.unity3d.com/cn/current/Manual/DrawCallBatching.html)（DrawCall）一次绘制（或渲染）**同一网格**的**多个副本**。它对于绘制诸如建筑物、树木和草地之类的在场景中重复出现的对象非常有用。

GPU 实例化在每次绘制调用时仅渲染相同的网格，但每个实例可以具有不同的参数（例如，颜色或比例）以增加变化并减少外观上的重复。

GPU 实例化可以降低每个场景使用的绘制调用数量。可以显著提高项目的渲染性能。

![GPU%20Instancing%20%EF%BC%88GPU%E5%AE%9E%E4%BE%8B%E5%8C%96%EF%BC%89%20832f70422ceb4fffbacf34bd7b4411cd/Untitled.png](渲染/GPU%20Instancing/Untitled.png)

未勾选Enable Instancing

![GPU%20Instancing%20%EF%BC%88GPU%E5%AE%9E%E4%BE%8B%E5%8C%96%EF%BC%89%20832f70422ceb4fffbacf34bd7b4411cd/Untitled%201.png](渲染/GPU%20Instancing/Untitled%201.png)

勾选Enable Instancing

![GPU%20Instancing%20%EF%BC%88GPU%E5%AE%9E%E4%BE%8B%E5%8C%96%EF%BC%89%20832f70422ceb4fffbacf34bd7b4411cd/Untitled%202.png](渲染/GPU%20Instancing/Untitled%202.png)

### 限制
- Unity 自动选取要实例化的网格渲染器组件和`Graphics.DrawMesh` 调用。请注意，**不支持 `SkinnedMeshRenderer`**。
- Unity 仅在单个 GPU 实例化绘制调用中批量处理那些共享相同网格和相同材质的游戏对象。**使用少量网格和材质可以提高实例化效率**。要创建变体，请修改着色器脚本为每个实例添加数据
- 还可以使用 `Graphics.DrawMeshInstanced` 和`Graphics.DrawMeshInstancedIndirect` 调用来通过脚本执行 GPU 实例化

### GPU 实例化可在以下平台和 API 上使用：
- Windows 上的 **DirectX 11** 和 **DirectX 12**
- Windows、macOS、Linux、iOS 和 Android 上的 **OpenGL Core 4.1+/ES3.0+**
- macOS 和 iOS 上的 **Metal**
- Windows、Linux 和 Android 上的 **Vulkan**
- PlayStation 4 和 Xbox One
- WebGL（需要 WebGL 2.0 API）

### 批处理优先级
进行批处理时，Unity 将==优先处理静态批处理==，然后==再处理实例化==。如果您将其中一个游戏对象标记为静态批处理，并且 Unity 成功对其进行批处理，则 Unity 会禁用该游戏对象的实例化，即使其渲染器使用实例化着色器也是如此。

某些因素可能会阻止游戏对象同时自动实例化。这些因素包括材质变化和深度排序。使用 `Graphics.DrawMeshInstanced` 可**强制 Unity 使用 GPU 实例化**来绘制这些对象。类似于 `Graphics.DrawMesh`，此函数为一帧绘制网格，不会创建不必要的游戏对象。

### GPU Skin
将骨骼动画转换为顶点动画，结合GPU Instancing完成大规模动画角色的渲染。

Unity蒙皮动画工作流程：

- CPU + SIMD
<p>CPU动画驱动骨骼->CPU骨骼驱动顶点->变换后的顶点发给GPU渲染</p>

- GPU Skinning
<p>CPU动画驱动骨骼->GPU上蒙皮->写回主存->变换后的顶点发给GPU渲染</p>

[GPU 实例化 - Unity 手册](http://docs.unity3d.com/cn/current/Manual/GPUInstancing.html)