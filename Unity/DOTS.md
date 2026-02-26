ECS是什么

|概念|传统 OOP (GameObject)|ECS (DOTS)|性能优势|
|---|---|---|---|
|**Entity**|GameObject（带 Transform 等）|纯 ID（int 或 Entity 类型），不带任何数据|极轻量|
|**Component**|MonoBehaviour 或 Component 类（带逻辑+数据）|纯数据 struct（IComponentData），无逻辑|内存连续、缓存友好|
|**System**|MonoBehaviour.Update() 等|SystemBase / ISystem，处理一组 Entity 的逻辑|可并行化、Burst 编译|
https://docs.unity3d.com/Packages/com.unity.entities@1.2/manual/ecs-workflow-intro.html

![[Pasted image 20260226231130.png]]


一个创作组件，是 MonoBehaviour 组件，可以保留你可以从编辑器传递给 ECS 组件的数值。
一个烘焙工具，将 ECS 组件附加到实体上，并用创作组件的值填充 ECS 组件。
![[Pasted image 20260226231608.png]]

C# Burst
提升速度5-100倍