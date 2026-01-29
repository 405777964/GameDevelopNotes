ECS是什么

|概念|传统 OOP (GameObject)|ECS (DOTS)|性能优势|
|---|---|---|---|
|**Entity**|GameObject（带 Transform 等）|纯 ID（int 或 Entity 类型），不带任何数据|极轻量|
|**Component**|MonoBehaviour 或 Component 类（带逻辑+数据）|纯数据 struct（IComponentData），无逻辑|内存连续、缓存友好|
|**System**|MonoBehaviour.Update() 等|SystemBase / ISystem，处理一组 Entity 的逻辑|可并行化、Burst 编译|
