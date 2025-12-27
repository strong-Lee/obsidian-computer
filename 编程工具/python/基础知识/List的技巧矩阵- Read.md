#### 1. ⚡ 极速：索引访问 (Indexing)

**场景**：获取第 i 个元素。
```python
data = ['a', 'b', 'c', 'd', 'e']
idx = 3

# ⚡ 极速 (Instant/Atomic) - O(1)
val = data[idx]  # 结果 'd'

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 寻址公式 (C Pointer Arithmetic):
#    - CPython 内部：`item = list->ob_item[idx]`
#    - 内存地址 = 数组首地址 + (索引 * 指针大小 8字节)
#    - 这是一个 CPU 指令级的操作，没有任何查找循环，也没有哈希计算。
#
# 2. 边界检查 (Bounds Check):
#    - 如果 `idx >= list->ob_size`，直接抛出 IndexError。
#    - PHP 会返回 NULL 或 抛出 Notice，Python 极其严格。
#
# 3. 性能对比：
#    - 比 PHP 的 `$arr[3]` 快很多，因为 PHP 即使是数字索引，底层可能还是走的 Hash 逻辑（取决于是否那是 packed array）。
```

#### 2. 🔰 基础：负数索引 (Negative Indexing)

**场景**：获取倒数第一个元素。  
**PHP思维**：$arr[count($arr) - 1] 或者 end($arr)。PHP 的 $arr[-1] 是查找 key 为 -1 的元素，而不是倒数！
```python
data = [10, 20, 30]

# ⚡ 极速 (Instant/Atomic)
last = data[-1] 

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 语法糖 (Syntactic Sugar):
#    - CPython 解释器发现索引是负数。
#    - 转换逻辑：`real_index = ob_size + negative_index`
#    - 比如 len=3, index=-1 -> 3 + (-1) = 2。
#    - 然后执行标准的数组访问 `ob_item[2]`。
#
# 2. 复杂度：
#    - 依然是严格的 O(1)。
```

#### 3. 🐢 较慢：切片读取 (Slicing)

**场景**：获取前 3 个元素。  
**PHP思维**：array_slice($arr, 0, 3)。
```python
nums = [0, 1, 2, 3, 4, 5]

# 🐢 较慢 (Slow) - O(K) K=切片长度
sub = nums[:3]  # 结果 [0, 1, 2]

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 内存分配 (Allocation):
#    - 切片**不是**视图 (View)！切片是**复制**！
#    - CPython 会创建一个全新的 list 对象，大小为 3。
#    - 这是一个 O(K) 的操作，涉及 malloc 和 引用计数增加。
#
# 2. 内存浪费警告 (#面试Tips):
#    - 如果 nums 是个 1GB 的大列表，`nums[:]` 会瞬间再消耗 1GB 内存（指针数组部分），
#      并触发所有元素的引用计数更新。
#    - 如果只是想遍历，**千万不要**用切片（如 `for x in nums[:3]:`），这会产生无意义的临时对象。
#    - 应该用 `itertools.islice` (迭代器) 来做零拷贝遍历。
```

#### 4. 🚀 进阶：序列解包 (Unpacking)

**场景**：把数组的前几个元素赋值给变量。  
**PHP思维**：list($a, $b) = $arr; 或者 $a = $arr[0];
```python
row = [100, 'UserA', True, 'Extra']

# 🚀 推荐 (Fast/Pythonic)
user_id, name, is_active, *meta = row

# 结果：
# user_id = 100
# meta = ['Extra']

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 操作码 (Opcode): `UNPACK_EX`
#    - 这是一个专门优化的指令。
#    - 它按顺序读取 `ob_item` 指针数组。
#    - `*meta` (Extended Iterable Unpacking) 会创建一个新列表，
#      将剩余的元素指针 copy 进去。
#
# 2. 架构思维：
#    - 这在处理 CSV 行解析或数据库返回结果时非常优雅。
#    - 比写 `user_id = row[0]; name = row[1]` 少了多次 opcode 查找。
```

#### 5. 🐢 较慢：线性查找 (Linear Search)

**场景**：判断一个值是否存在于列表中。  
**PHP思维**：in_array($val, $arr)。
```python
users = [101, 102, 103, ... , 99999]
target = 50000

# 🐢 较慢 (Slow) - O(N)
exists = target in users

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 迭代比较 (Sequential Scan):
#    - CPython 执行 `PySequence_Contains`。
#    - 它是一个 `for` 循环：从 `ob_item[0]` 开始，逐个拿出对象。
#    - 对每个对象调用 `PyObject_RichCompareBool(item, target, Py_EQ)`。
#    - 这相当于调用了对象的 `__eq__` 方法。
#
# 2. 性能陷阱：
#    - 如果列表有一百万个元素，且目标在最后，它就要做一百万次比较。
#
# 3. 架构优化 (#面试Tips):
#    - 如果你需要频繁查找，**必须**把 List 转为 Set (`set(users)`)。
#    - Set 底层是哈希表，查找是 O(1)。
#    - PHP 的数组自带哈希索引，所以 `$arr['key']` 很快，但 `in_array` 也是 O(N)。
```

#### 6. 🚀 推荐：二分查找 (Binary Search)

**场景**：在一个**有序**的列表中查找位置。  
**PHP思维**：很少有人手动写二分查找，通常依赖数据库查询。
```python
import bisect
scores = [60, 70, 80, 90, 95] # 必须是有序的

# 🚀 推荐 (Fast/Pythonic) - O(log N)
# 动作：查找 75 应该插入的位置，或者判断是否存在
idx = bisect.bisect_left(scores, 75) 
# 结果 idx = 2 (70 和 80 之间)

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. C 模块实现:
#    - `bisect` 模块完全由 C 语言编写。
#    - 它避免了在 Python 层面写 `while low <= high` 的循环开销。
#    - 它直接在 C 指针数组上进行二分跳跃。
#
# 2. 适用性：
#    - 只适用于**已排序**的列表。
#    - 这是 List 这种连续内存结构的优势（随机访问 O(1) 配合二分查找）。
```

#### 7. 🚀 进阶：步长切片 (Stride Slicing)

**场景**：获取列表中所有偶数位索引的元素 (0, 2, 4...)。  
**PHP思维**：for ($i=0; $i < count($arr); $i+=2).
```python
data = ['a', 'b', 'c', 'd', 'e', 'f']

# 🚀 推荐 (Fast/Pythonic)
evens = data[::2] # ['a', 'c', 'e']

# 🐢 较慢 (Slow) - Python 循环
# evens = []
# for i in range(0, len(data), 2):
#    evens.append(data[i])

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 切片对象 (Slice Object):
#    - `[::2]` 创建了一个 `slice(0, max, 2)` 对象。
#
# 2. 内存动作：
#    - CPython 预先计算出新列表的大小 (len / 2)。
#    - `PyList_New` 分配内存。
#    - 这是一个 C 语言层面的紧凑循环：`src_ptr += step`，然后复制指针。
#    - 比 Python 解释器里的 for 循环指令快得多。
```