#### [Tips 1] 字面量 {} vs dict()：谁才是亲儿子？

**场景**：初始化一个空字典或包含常量的字典。
**PHP思维**：PHP 中 array() 和 [] 几乎没区别，都是语法糖。
```python
# 🚀 推荐 (Fast/Pythonic)
# 直接使用字面量语法
my_conf = {"host": "localhost", "port": 3306} 

# 🐢 较慢 (Slow)
# 使用类构造函数
my_conf_slow = dict(host="localhost", port=3306)

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：
#    - `{}` (Literal): 编译器直接将其视为 BUILD_MAP 字节码指令。
#    - `dict()` (Class): 编译器将其视为 CALL_FUNCTION，需要进行函数调用栈的压栈、参数解析、创建对象等操作。
# 2. 底层原理 (CPython VM):
#    - `{}` 触发 `_PyDict_NewPresized(size)`。C 语言层面直接根据元素数量 malloc 内存。
#    - `dict()` 涉及 `type_call` -> `dict_new` -> `dict_init`，路径长得多。
# 3. 内存动作：
#    - `{}`: 分配一个 PyDictObject 结构体，并预分配好存放 Key/Value 指针的数组（dk_entries）。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：在高频循环（如每秒 10万次）中创建字典时，`dict()` 的函数调用开销会累积。
# 2. 替代方案：始终使用 `{}`。
# 3. 权衡：`dict(key=val)` 写法对 Key 为字符串时比较简洁（不用写引号），但为了性能应牺牲这点语法糖。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：为什么 `{}` 比 `dict()` 快？快多少？
# 策略：不要只说“快”，要说“字节码”。“`{}` 是编译器层面的指令优化（BUILD_MAP），是原子操作；而 `dict()` 是一次完整的 Python 函数调用，涉及栈帧开销。通常快 2 倍左右。”
```

#### [Tips 2] 哈希性 (`Hashability`)：PHP 程序员最大的坑

**场景**：想用一个列表（如坐标 [x, y]）作为字典的 Key。
**PHP思维**：PHP 数组的 Key 会自动强转为 Int 或 String。你甚至可以用数组做 Key（虽然会报错或转为 "Array" 字符串）。
```python
# 💀 性能杀手 (Killer) / 运行时崩溃
# broken_dict = {[1, 2]: "value"}  # ❌ TypeError: unhashable type: 'list'

# 🚀 推荐 (Fast/Pythonic)
# 使用 Tuple (元组) 作为 Key
valid_dict = {(1, 2): "value"} 

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：Python 在插入 Dict 时，必须计算 Key 的 hash 值。
# 2. 底层原理 (PyObject_Hash):
#    - Dict 依赖 `hash(key)` 来定位槽位。
#    - List 是可变的 (Mutable)。如果 List 的内容变了，hash 值就会变，导致无法再次找到该元素。因此 CPython 强制禁止 List 定义 `__hash__`。
#    - Tuple 是不可变的 (Immutable)，其 Hash 值是基于内容的固定值。
# 3. 内存动作：
#    - CPython 调用 `PyObject_Hash(key)`。对于 Tuple，它会通过算法（xxHash 或类似）混合内部元素的 Hash 值生成最终的 `long` 类型 Hash。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果 Key 是复杂的对象，计算 Hash 可能会很慢（O(N) 复杂度，N为Key的长度）。
# 2. 替代方案：自定义类，实现高效的 `__hash__`，或者使用简单的 ID (Int/Str) 作为 Key。
# 3. 权衡：使用对象做 Key 方便业务逻辑，但增加了哈希计算开销。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：Python 中什么能做 Key？为什么 List 不行？
# 策略：抛出关键词“可变性 (Mutability)”。“只有不可变对象（Immutable）才是 Hashable 的。List 可变，如果允许做 Key，修改 List 后 Hash 值改变，这个 Key 就‘失联’了，破坏了哈希表的完整性。”
```

#### [Tips 3] `fromkeys` 的“幽灵引用”陷阱

**场景**：初始化一个类似 PHP array_fill_keys 的字典，默认值为列表。
**PHP思维**：PHP 复制的是值，互不干扰。Python 复制的是引用（指针）。
```python
# 💀 性能杀手 (Killer) / 逻辑炸弹
keys = ['u1', 'u2', 'u3']
# 这里的 [] 是同一个列表对象！
bad_matrix = dict.fromkeys(keys, []) 
bad_matrix['u1'].append(99) 
# 结果：{'u1': [99], 'u2': [99], 'u3': [99]} -> 所有人都改了！

# 🚀 推荐 (Fast/Pythonic)
# 使用字典推导式 (Dict Comprehension)
good_matrix = {k: [] for k in keys}
good_matrix['u1'].append(99)
# 结果：{'u1': [99], 'u2': [], 'u3': []} -> 正常

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：`fromkeys` 接收一个 `value` 参数。
# 2. 底层原理 (Pointer Copying):
#    - CPython 在 C 层面循环遍历 keys，将同一个 `PyObject *value` 指针赋值给字典的每一个 Entry。
#    - 这里没有发生 `malloc` 来创建新列表，只是引用计数 `ob_refcnt` 增加了 N 次。
# 3. 内存动作：
#    - 所有 Key 的 Value 指针指向内存地址 0xDEADBEEF (假设是那个空列表)。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果 Value 是不可变对象（如 None, 0），`fromkeys` 是极速的 O(N)。只有 Value 可变时才是坑。
# 2. 替代方案：字典推导式 `{k: list() for k in keys}`。
# 3. 权衡：推导式稍微慢一点点（因为要多次调用 `list()` 构造函数），但保证了逻辑正确。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：dict.fromkeys(keys, [1]) 有什么问题？
# 策略：画图解释。“这叫浅拷贝陷阱。fromkeys 仅仅广播了指针，没有深拷贝对象。对于 Mutable 对象，必须使用推导式来为每个 Key 实例化新的对象。”
```

#### [Tips 4] 字典推导式 (Dict Comprehension)

**场景**：从另一个数据结构转换生成字典（例如：ID列表 -> ID:Name 映射）。
**PHP思维**：`foreach` 循环，然后 `$arr[$k] = $v`。
```python
users = [('id1', 'Alice'), ('id2', 'Bob')]

# 🐢 较慢 (Slow) - 传统的 PHP 风格
d = {}
for k, v in users:
    d[k] = v

# 🚀 推荐 (Fast/Pythonic)
d = {k: v for k, v in users}

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：编译器将推导式优化为特殊的字节码循环。
# 2. 底层原理 (MAP_ADD):
#    - `for` 循环版本：每次循环涉及 `STORE_SUBSCR` 字节码，会触发 Python 栈帧检查。
#    - 推导式版本：在 C 语言层面运行一个紧凑的循环，使用 `MAP_ADD` 指令直接操作字典结构体，跳过了一些 Python 层面的开销。
# 3. 内存动作：
#    - 类似于 `list_append` 的优化，推导式内部能更智能地根据输入长度（如果可知）预估字典大小（Hinting），减少 `realloc`（扩容）次数。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：数据量极大时（100w+），内存是瓶颈。
# 2. 替代方案：如果只是为了查一次，考虑不生成字典，直接用 Generator 遍历。
# 3. 权衡：空间换时间。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：推导式只是语法糖吗？
# 策略：“不仅仅是。它在字节码层面是不同的（LIST_APPEND / MAP_ADD），拥有 C 级别的循环优化，通常比手动 Python `for` 循环快 10%-20%。”
```

#### [Tips 5] 紧凑型字典 (Compact Dict) —— Python 3.6+ 的黑魔法

**场景**：理解为什么 Python 字典现在是有序的，且比 PHP 数组省内存。
**PHP思维**：PHP 数组为了有序，维护了一个双向链表，内存开销巨大。
```python
# 🏴‍☠️ 黑客/底层 (Hacker/Internals)
d = {"name": "Alex", "age": 20, "job": "Dev"}

# 在 Python 3.6 之前 (类似 PHP): 
# 它是稀疏数组 (Sparse Array)，大量的内存空洞。
# Entries: [hash|k|v, null, null, hash|k|v, null ...] 

# 在 Python 3.6+ (Compact):
# ⚡ 极速 (Instant) - 内存布局改变
# 1. Indices (索引数组):  [None, 0, None, 1, 2, None...] (只存 offset，非常小)
# 2. Entries (实体数组):  [[hash, k, v], [hash, k, v], [hash, k, v]] (密集存储)

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：分离索引和数据。
# 2. 底层原理 (dk_indices vs dk_entries):
#    - `Indices` 数组是 `int8` 或 `int16` 类型的数组，只存储它在 `Entries` 数组中的下标。
#    - `Entries` 数组严格按照插入顺序追加 (Append only)。
# 3. 内存动作：
#    - 这种布局节省了 20%~25% 的内存，因为 `Indices` 数组非常小，而 `Entries` 没有空洞。
#    - 遍历字典时，直接线性扫描 `Entries` 数组，利用了 CPU 缓存行 (Cache Line)，极快。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果频繁删除元素，Compact Dict 需要标记删除（Dummy），可能导致 `Entries` 数组中存在“尸体”，触发 resize。
# 2. 替代方案：无。这是 CPython 内置机制。
# 3. 权衡：插入和查找变快，内存变小，删除稍微复杂一点（逻辑删除）。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：Python 3.7 以后字典为什么有序了？
# 策略：“这是 Compact Dict 架构带来的副作用（Feature）。数据是按插入顺序 append 到 Entries 数组的，而查找是通过 Indices 跳转的。这比 PHP 维护双向链表要高效得多，既省内存又对 CPU Cache 友好。”
```

#### [Tips 6] 扩容机制 (Resizing) —— 避免性能抖动

**场景**：一次性向字典插入大量数据。
**PHP思维**：PHP 数组扩容也是自动的，通常翻倍。
```python
import sys

# 🚀 推荐 (Fast/Pythonic)
# 如果预知大小，一次性构建比由小变大好
data = {i: i for i in range(1000)}

# 🐢 较慢 (Slow)
d = {}
for i in range(1000):
    # 可能会触发多次 Resize (realloc + rehash)
    d[i] = i

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：字典的 Load Factor (负载因子) 限制为 2/3 (0.66)。
# 2. 底层原理 (USABLE_FRACTION):
#    - 当 `active_entries >= table_size * 2/3` 时，CPython 触发 `dictresize`。
#    - 新大小通常是 `current_used * 3` (对于小字典) 或 `* 2` (大字典)，必须是 2 的幂次方。
# 3. 内存动作 (Expensive!):
#    - 申请一块更大的内存。
#    - **关键**：所有的 Key 必须重新 Hash，或者利用缓存的 hash 值重新计算在新表中的位置 (Re-insertion)。这是一个 O(N) 操作。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果你在一个循环里 `d[k]=v` 插入 100万条数据，会触发多次全量 Rehash，导致程序出现周期性卡顿。
# 2. 替代方案：
#    - 使用 `dict(sequence)` 一次性构建，CPython 会尝试预估大小。
#    - 如果必须循环，甚至可以考虑先用 list 收集，最后转 dict。
# 3. 权衡：内存峰值 vs CPU 时间。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：字典扩容时发生了什么？
# 策略：“会发生 Resize。不仅仅是 malloc 新内存，最耗时的是所有现存元素都需要搬家（Re-insertion），重新计算在新 Indices 数组中的位置。所以预分配大小（如果可能）是优化的关键。”
```

#### [Tips 7] 字典合并 (Merging) —— Python 3.9 的新语法糖

**场景**：合并两个配置项（默认配置 + 用户配置）。
**PHP思维**：$merged = array_merge($default, $user) 或 $default + $user。
```python
default_conf = {"host": "localhost", "port": 3306}
user_conf = {"port": 8080, "debug": True}

# 🐢 较慢 (Slow) - Python 3.5 以前的老写法
# 需要创建中间副本，甚至多次拷贝
merged = default_conf.copy()
merged.update(user_conf) 

# 🚀 推荐 (Fast/Pythonic) - Python 3.9+ 
# 使用联合操作符 Union Operator (|)
final_conf = default_conf | user_conf
# 结果: {'host': 'localhost', 'port': 8080, 'debug': True}

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：创建新字典，并将两个字典的 Entry 批量复制进去。
# 2. 底层原理 (dict_merge):
#    - 这是一个 O(N + M) 的操作。
#    - CPython 会先计算总大小，预分配足够内存（避免中间扩容），然后直接进行内存块的复制（如果是 Compact 结构，拷贝效率很高）。
# 3. 内存动作：
#    - 申请一个新的 PyDictObject。
#    - 只有指针被复制（Shallow Copy）。如果 Value 是可变对象，两个字典仍然共享该对象。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果是在高频循环中合并大字典，频繁 malloc 新字典是杀手。
# 2. 替代方案：`collections.ChainMap`。它不创建新字典，而是创建一个“逻辑视图”，查找时依次查找。O(1) 创建。
# 3. 权衡：`ChainMap` 读取速度稍慢（因为要遍历链表），但写入/创建极快。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：`d1 | d2` 和 `d1.update(d2)` 有什么区别？
# 策略：强调“原地 (In-place) vs 新建”。“`update` 是原地修改 `d1`，没有返回值；`|` 运算符会 malloc 一个全新的字典，不影响原数据。函数式编程推荐用 `|` 保持数据不可变性。”
```
#### [Tips 8] `defaultdict vs PHP Auto-vivification`

**场景**：统计单词出现位置，或者构建嵌套结构。
**PHP思维**：`$arr['user']['id'] = 1`。PHP 会自动创建不存在的层级（Auto-vivification），这是 PHP 最大的魔力。Python 默认会报 KeyError。
```python
from collections import defaultdict

# 💀 性能杀手 (Killer) / 代码冗余
d = {}
key = "users"
if key not in d:      # 1. 查询
    d[key] = []       # 2. 插入 (如果不存在)
d[key].append(101)    # 3. 再次查询 + 插入

# 🚀 推荐 (Fast/Pythonic)
# 告诉字典：如果 Key 不存在，就调用 list() 创建一个新列表给我
dd = defaultdict(list)
dd["users"].append(101) 
# 结果: {'users': [101]}

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：访问不存在的 Key 时，自动触发工厂函数。
# 2. 底层原理 (__missing__):
#    - 当 `PyDict_GetItem` 失败时，`defaultdict` 重写了 `__missing__` 方法。
#    - 它会调用构造时传入的 `default_factory` (这里是 `list`)，生成新对象，并将其插入字典，然后返回该对象。
# 3. 复杂度：
#    - 依然是 O(1)，但比单纯的 GetItem 多了一次函数调用（调用 list()）。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：`defaultdict` 在序列化（JSON）时可能会有麻烦，因为它不是标准 dict。
# 2. 替代方案：在导出数据时使用 `dict(dd)` 转回普通字典。
# 3. 权衡：开发效率极大提升，性能损耗几乎可以忽略。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：如何实现一个无限嵌套的字典（像 PHP 那样）？
# 策略：“可以使用递归的 lambda：`tree = lambda: defaultdict(tree)`。这样 `d['a']['b']['c']` 就会自动创建。虽然很酷，但在 Python 中不建议过度使用，因为隐式创建可能会掩盖逻辑错误。”
```

#### [Tips 9] setdefault 的原子性操作

**场景**：初始化字典中的某个 Key，如果它不存在的话。但如果存在则获取它。
**PHP思维**：`if (!isset($arr[$k])) $arr[$k] = val; return $arr[$k];`
```python
d = {}

# 🐢 较慢 (Slow) - 两次 Hash 查找
# 1. Check, 2. Set, 3. Get (implicit)
if "k" not in d:
    d["k"] = []
d["k"].append(1)

# 🚀 推荐 (Fast/Pythonic) - 一次 Hash 查找
# "Get this key, or set it to default and return that"
# 这一行在 C 层面是原子的（针对字典操作而言）
d.setdefault("k", []).append(1)

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：`setdefault` 在 C 层面只执行一次查找逻辑。
# 2. 底层原理 (dict_setdefault):
#    - 它计算 hash，查找 slot。
#    - 如果 slot 为空，直接填入 default 值并返回。
#    - 如果 slot 有值，直接返回该值。
#    - 避免了 Python 层面 `if` 判断带来的字节码跳转开销。
# 3. 坑点 (Performance Pitfall):
#    - `setdefault("k", calculate_heavy_default())`。注意！即便 key 存在，`calculate_heavy_default()` 也会被执行！因为参数是在函数调用前计算的。这点不如 `defaultdict` 懒加载。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 替代方案：如果 default 值创建开销很大（如大对象），老老实实写 `if key not in d`，或者用 `defaultdict`。
# 2. 权衡：`setdefault` 适合默认值是简单字面量（None, [], 0）的情况。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：setdefault 和 defaultdict 怎么选？
# 策略：“如果是为了构建聚合字典，`defaultdict` 性能更好且代码更干净。`setdefault` 适合临时的、单次的缺省值填充，或者当你无法控制字典的创建过程时（比如字典是传参进来的）。”
```

#### [Tips 10] 计数器优化 (Counter)

**场景**：统计列表中元素出现的频率。
**PHP思维**：`$counts = []; foreach ($items as $i) $counts[$i]++;`
```python
from collections import Counter
items = ['apple', 'banana', 'apple', 'orange']

# 🐢 较慢 (Slow) - 手动循环
counts = {}
for item in items:
    counts[item] = counts.get(item, 0) + 1

# 🚀 推荐 (Fast/Pythonic)
# C 语言加速的统计
c = Counter(items)
# 结果: Counter({'apple': 2, 'banana': 1, 'orange': 1})

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：`Counter` 是 dict 的子类，它的构造函数调用了 `_count_elements`。
# 2. 底层原理 (C Accelerator):
#    - 如果输入是 List，CPython 内部有一个专门的 C 函数 `_count_elements`，它绕过了 Python 层面的 `__getitem__` 和 `__setitem__` 开销，直接操作底层结构。
# 3. 性能：
#    - 比手写的 Python 循环通常快 2 倍以上。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果你只需要 Top K（比如前 10 名），Counter 有现成的 `most_common(k)`，利用了堆排序（Heap Queue）算法，比全量排序 `sorted()` 快得多。
# 2. 替代方案：无，这是标准库最优解。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：如何合并两个计数结果？
# 策略：“Counter 支持加法运算：`c1 + c2`。底层会自动合并计数，非常方便用于 MapReduce 类型的任务。”
```

#### [Tips 11] 冲突解决 (Collision Resolution) —— 开放寻址法

**场景**：理解为什么 Python 字典比 PHP 数组更怕“坏的 Hash 函数”。
🏴‍☠️ 黑客/底层 (Hacker/Internals) —— **核心原理差异**
```python
# 假设我们强行制造 Hash 冲突 (仅作演示，Python 内部有随机化)
# PHP 采用 "链地址法 (Chaining)"：冲突了就挂链表。
# Python 采用 "开放寻址法 (Open Addressing)"：冲突了就找下一个空位。

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：当 `hash("a")` 和 `hash("b")` 映射到同一个 slot 时。
# 2. 底层原理 (Linear Probing with Perturbation):
#    - PHP: Bucket[Hash] -> Entry -> Entry (链表)。即使冲突多，只是链表变长。
#    - Python: 没有链表。如果 Slot i 被占了，根据公式 `j = (5*j) + 1 + perturb` 寻找下一个位置。
# 3. 内存动作：
#    - 这就是为什么 Python 字典必须保持至少 1/3 的空闲 slots。
#    - 如果冲突太剧烈，插入元素会变成 O(N)！Python 必须在数组中不断跳跃寻找空位 (Probing)。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：HashDoS 攻击。如果攻击者构造大量相同 Hash 的 Key，Python 字典会退化成线性查找，CPU 100%。
# 2. 保护机制：Python 3.4+ 默认启用了 SipHash 算法，并在每次启动时随机化 Hash Seed。
# 3. 经验总结：永远不要在 Python 中自定义简单的 `__hash__`，除非你非常懂分布算法。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：为什么 Python 不用链表解决哈希冲突？
# 策略：“为了利用 CPU 缓存（Cache Locality）。开放寻址法的数据都在连续内存中，遍历和查找时缓存命中率极高。而链表指针跳转会导致大量的 Cache Miss。这是 Python 追求极致速度的体现。”
```

#### [Tips 12] 只读字典 (MappingProxy) —— 打造不可变配置

**场景**：你希望你的全局配置字典是只读的，防止被某个小白实习生无意修改。
**PHP思维**：封装在 Class 里用 private 属性，或者 define 常量数组 (PHP 7+)。
```python
from types import MappingProxyType

# 原始数据
_config = {"db": "mysql", "pwd": "123"}

# 🚀 推荐 (Fast/Pythonic)
# 创建一个只读视图
public_config = MappingProxyType(_config)

# public_config["db"] = "pg"  
# ❌ TypeError: 'mappingproxy' object does not support item assignment

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：创建一个代理对象，拦截所有写入操作。
# 2. 底层原理 (Proxy Pattern):
#    - `MappingProxyType` 在 C 层面并不复制数据，它只是持有原字典的指针 (`PyObject *`).
#    - 它的 `mp_subscript` (读取) 转发给原字典。
#    - 它的 `mp_ass_subscript` (写入) 只有一行代码：抛出 TypeError。
# 3. 内存动作：
#    - 极轻量级，几乎没有内存开销。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 场景：类内部的 `__dict__` 实际上就是通过这种机制暴露为 `MyClass.__dict__` 的（你是改不掉类属性字典本身的引用的，只能改内容）。
# 2. 权衡：注意，如果原字典 `_config` 变了，`public_config` 也会跟着变（因为是透视镜，不是快照）。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：如何实现完全不可变的字典？
# 策略：“MappingProxyType 是视图不可变。如果需要真正的逻辑不可变，可以使用 Hashable 的 `frozendict` (需第三方库) 或者将数据转为 `NamedTuple`。”
```