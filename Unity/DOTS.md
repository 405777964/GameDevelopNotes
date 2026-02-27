ECS是什么

|概念|传统 OOP (GameObject)|ECS (DOTS)|性能优势|
|---|---|---|---|
|**Entity**|GameObject（带 Transform 等）|纯 ID（int 或 Entity 类型），不带任何数据|极轻量|
|**Component**|MonoBehaviour 或 Component 类（带逻辑+数据）|纯数据 struct（IComponentData），无逻辑|内存连续、缓存友好|
|**System**|MonoBehaviour.Update() 等|SystemBase / ISystem，处理一组 Entity 的逻辑|可并行化、Burst 编译|
https://docs.unity3d.com/Packages/com.unity.entities@1.2/manual/ecs-workflow-intro.html

![[Pasted image 20260226231130.png]]


一个创作组件 Authoring，是 MonoBehaviour 组件，可以保留你可以从编辑器传递给 ECS 组件的数值。
一个烘焙工具 Baker，将 ECS 组件附加到实体上，并用创作组件的值填充 ECS 组件。
![[Pasted image 20260226231608.png]]
烘焙是离线完成的，在游戏运行前就已经完成了生成了一个包含Spawner组件的Entity，所以System可以找到Spawner

System生成实体
![[Pasted image 20260226234620.png]]
组件类型

共享组件
清理组件

![[Pasted image 20260227164553.png]]
![[Pasted image 20260227164446.png]]
Execute方法会查找同时拥有 `LocalTransform`、`PostTransformMatrix` 和 `RotationSpeed` 组件的实体，并为它们添加 `ExecuteIJobEntity` 组件。

而System里的RequireForUpdate会等到场景里存在ExecuteIJobEntity才执行，避免update空转
- 当系统进入更新循环，对于设置了 `RequireForUpdate<ExecuteIJobEntity>();` 的 `RotationSystem`，在执行 `OnUpdate` 之前，框架并不会先去检查是否有 `ExecuteIJobEntity` 组件。而是直接进入 `OnUpdate` 方法。
- 在 `OnUpdate` 方法中调度 `RotateAndScaleJob` 时，框架会立即根据 `RotateAndScaleJob` 的 `Execute` 方法参数确定查询条件，查找场景（包括子场景）中符合条件的实体，并为这些实体添加 `ExecuteIJobEntity` 组件。这个添加操作几乎是瞬间完成的，在同一更新周期内，紧接在调度之后。


![[Pasted image 20260227174002.png]]
在 Unity ECS（Entity - Component - System）框架中，`IJobChunk` 和 `IJob` 是两种不同类型的作业，它们在功能、使用场景和性能特点等方面存在明显区别。

### 1. 数据处理方式

- **IJobChunk**：
    
    - **以数据块为单位**：`IJobChunk` 作业以 `ArchetypeChunk` 为处理单元。`ArchetypeChunk` 是一组具有相同组件类型集合的实体，这意味着 `IJobChunk` 会一次性处理多个实体的数据。例如，假设场景中有大量具有 `LocalTransform` 和 `Health` 组件的实体，这些实体可能被分组到多个 `ArchetypeChunk` 中，`IJobChunk` 作业会逐个处理这些数据块中的所有实体。
    - **高效内存访问**：由于同一 `ArchetypeChunk` 内的实体组件数据在内存中连续存储，`IJobChunk` 利用了数据局部性原理，CPU 能够更高效地从内存中读取和写入数据，减少缓存未命中的情况，提升性能。
    
- **IJob**：
    
    - **单一任务处理**：`IJob` 通常用于执行简单的、单一的任务，并不直接与实体的组件数据块相关联。它一般处理独立的计算或操作，例如对单个变量进行复杂的数学运算，或者执行一些与 ECS 实体组件关系不大的逻辑。
    - **无特定数据结构绑定**：`IJob` 没有像 `IJobChunk` 那样与 `ArchetypeChunk` 这种特定的数据结构紧密绑定，它的输入和输出可以是各种类型的数据，包括基本数据类型、自定义结构体等。
    

### 2. 并行处理能力

- **IJobChunk**：
    
    - **高度并行**：设计初衷就是为了利用多核 CPU 进行并行处理。因为它以数据块为单位处理实体，非常适合在多个 CPU 核心上同时处理不同的数据块，从而实现对大量实体的高效并行操作。例如，在一个大规模的角色扮演游戏中，使用 `IJobChunk` 可以并行处理众多角色的移动、攻击等行为。
    - **自动负载均衡**：ECS 框架会自动对 `IJobChunk` 作业进行负载均衡，将不同的 `ArchetypeChunk` 分配到不同的 CPU 核心上执行，充分利用系统资源。
    
- **IJob**：
    
    - **有限并行**：`IJob` 本身并非专为大规模并行处理实体设计。虽然也可以通过一些方式并行执行，但相比 `IJobChunk`，其并行处理能力较弱。例如，在调度 `IJob` 作业时，通常只能在单个 CPU 核心上执行，除非通过特定的方式将多个 `IJob` 实例并行化，但这种并行化的粒度和效率都不如 `IJobChunk`。
    - **适合简单并行任务**：`IJob` 更适合那些不需要处理大量实体数据，但又希望利用一定并行性的简单任务，比如并行计算一些独立的数值结果。
    

### 3. 使用场景

- **IJobChunk**：
    
    - **大规模实体处理**：当需要对大量具有相同组件类型组合的实体进行相同操作时，`IJobChunk` 是理想的选择。例如，在一个策略游戏中，对地图上大量士兵的位置更新、状态检查，或者在一个粒子系统中对大量粒子的属性更新等场景。
    - **ECS 组件密集型操作**：如果操作涉及到频繁访问和修改 ECS 实体的组件数据，`IJobChunk` 能够通过高效的数据访问方式提升性能。比如，同时对多个敌人的生命值、攻击力等组件数据进行更新。
    
- **IJob**：
    - **简单独立任务**：适用于执行简单的、与 ECS 实体组件关系不大的独立任务。例如，计算游戏中的一些全局参数，如游戏难度系数的动态调整，或者进行一些初始化数据的预处理操作。
    - **非性能敏感任务**：当任务的执行时间较短，对性能要求不是特别高，且不需要大规模并行处理时，可以使用 `IJob`。例如，在游戏启动时执行一些简单的配置加载操作。


C# Burst
提升速度5-100倍