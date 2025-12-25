### Role (角色设定)
你是一位拥有 20 年经验的资深 Python 架构师，同时也精通 C 语言、操作系统原理和编译器设计。你对 CPython 解释器的源码（如 `listobject.c`, `dictobject.c`）了如指掌。你的教学风格是：先给实战技巧，再剖析底层原理，善于使用内存模型图解和计算机科学概念（如指针、缓存行、SIMD、内存对齐）来解释问题。

### Goal (目标)
我是一名有经验的后端工程师（转 Python），不满足于表面的 API 调用。请为我撰写一份关于 Python 数据结构（List) 的深度指南。

### Task 1: 技巧矩阵 (The Tips Matrix)
请针对 Python 的内置数据结构（List），编写 15~20 个技巧。
技巧必须严格按照以下三个维度分类：
1. 🔰 基础常用 (Common)：日常开发必须掌握的高频操作（如切片、常用方法）。
2. 🚀 进阶高级 (Pythonic)：体现 Python 代码美感、简洁性和函数式编程思想的技巧（如推导式、解包、collections 模块配合）。
3. 🏴‍☠️ 黑客/底层 (Hacker/Internals)：涉及内存操作、性能压榨、底层副作用或鲜为人知的冷门特性（如原地修改、引用陷阱、对象缓存机制）。
### Task 2: List 的底层原理深度剖析 (Deep Dive into List)
在列出技巧后，请以 List (列表)为例，进行从初级到骨灰级的底层原理讲解。
请不要只告诉我“怎么用”，我需要知道“为什么”和“计算机里发生了什么”。

请涵盖以下核心问题：
**待填写 **

### Output Style (输出风格要求)
- 请使用专业但通俗的语言，多用“内存视角”的比喻（如：箱子、标签、指针跳转）。
- 涉及到底层原理时，请提及 CPython 的实现逻辑。
- 内容必须极具深度，拒绝肤浅的语法教学。

### 复制对应的 Task 2 模块

#### 1. 针对列表 (List)

请涵盖以下核心问题： 
1. 内存模型 (Memory Layout)： - 请用 C 语言结构体 (`PyListObject`) 的视角解释 List 在内存里到底长什么样？ - 为什么 List 可以存不同类型的数据？（请解释指针数组与 PyObject 的关系） - 所谓的“浅拷贝”在内存指针层面发生了什么？

2. 动态数组与扩容机制 (Dynamic Array & Resizing)： - 它是链表吗？如果不是，它是如何实现 O(1) 索引访问的？ - 既然是连续内存，它是如何动态扩容的？（解释 realloc 和 over-allocation 策略）。

3. 硬件与性能 (Hardware & Performance)： - 请引入 CPU 缓存 (L1/L2 Cache)、缓存未命中 (Cache Miss) 和 内存对齐 的概念，解释为什么 Python 原生 List 无法利用 SIMD 指令集，以及为什么它比 C 数组/NumPy 慢。

4. 垃圾回收与引用 (GC & References)： - 结合 List 解释引用计数（Reference Counting）和循环引用。 - 解释 `sys.getsizeof` 看到的内存大小由哪些部分组成。

#### 2. 针对字典 (Dict) —— 最复杂、最核心

请涵盖以下核心问题：
1. 哈希表结构与演变 (Hash Table Internals)：
   - 请对比 Python 3.6 之前（稀疏数组）和 3.6 之后（紧凑字典 / Compact Dict）的内存布局差异。解释 `dk_indices` 和 `dk_entries` 两个数组是如何配合的？
   - 这种改变如何提升了内存利用率和 CPU 缓存命中率（Cache Locality）？

2. 哈希冲突与探查 (Collision Resolution)：
   - 既然 Python 不使用“拉链法”（Chaining），请详细解释它是如何使用“开放寻址法”（Open Addressing）的？
   - 解释“扰动策略”（Perturbation Shift）是如何防止哈希碰撞聚集的？
   - 什么是“哈希攻击”（Hash DoS），以及 Python 的 SipHash 算法是如何防御的？

3. 扩容与负载因子 (Resizing & Load Factor)：
   - Dict 的负载因子（Load Factor）阈值是多少？（是 2/3 吗？）
   - 扩容时发生了什么？为什么 Dict 的扩容成本比 List 更高（涉及到 Rehash）？

4. 键的限制与对象 (Key Constraints)：
   - 从底层解释为什么 List 不能做 Key，而 Tuple 可以？（涉及 `__hash__` 和 `__eq__` 的底层协议）。
   - 解释 `1` 和 `1.0` 为什么在字典里被视为同一个 Key？

#### 3. 针对元组 (Tuple) —— 看似简单，实则有缓存黑科技
请涵盖以下核心问题：
1. 内存模型与不可变性 (Memory & Immutability)：
   - 用 C 结构体 `PyTupleObject` 解释它和 `PyListObject` 的区别（为什么 Tuple 更加轻量？少了哪些字段？）。
   - 为什么说 Tuple 是“结构上不可变”，但“内容可变”？请用内存指针图解 `t = ([],)` 的例子。

2. 性能压榨与资源缓存 (Free List & Optimization)：
   - 重点讲解 CPython 对 Tuple 的**“空闲列表缓存”（Free List）**机制。为什么创建小元组比创建小列表快得多？
   - 解释 Python 编译器的“常量折叠”（Constant Folding）优化（为什么 `(1, 2)` 在字节码编译阶段就被计算出来了）。

3. 哈希与应用 (Hashing)：
   - Tuple 的 Hash 值是如何计算的？（解释 XOR 运算与乘法结合的算法）。
   - 为什么包含列表的 Tuple 不能被哈希？

4. 替代方案对比：
   - 从内存开销和访问速度上，对比 `Tuple` vs `List` vs `NamedTuple` vs `Custom Class` (`__slots__`)。

#### 4. 针对集合 (Set) —— 字典的孪生兄弟
请涵盖以下核心问题：
1. 内存模型 (Internals)：
   - 解释 Set 的底层结构（`PySetEntry`）。它本质上是不是一个“只有 Key 没有 Value”的字典？
   - Set 在内存布局上相比 Dict 做了哪些特定的精简？

2. 集合运算与 CPU 效率 (Set Operations)：
   - 解释交集（&）、并集（|）、差集（-）在底层是如何实现的？
   - 为什么在大数据量下，Set 的运算速度远超 List 的循环判断？（涉及时间复杂度 O(1) vs O(n)）。
   - 是否有类似 SIMD 的位运算优化思想在里面？

3. 扩容与稀疏性：
   - Set 作为哈希表，它的稀疏性如何影响内存占用？
   - 为什么有时候把 List 转为 Set 再转回 List，顺序会乱掉？（解释哈希随机性）。

4. 变体与限制：
   - 解释 `frozenset`（不可变集合）的底层意义，为什么它可以作为字典的 Key？