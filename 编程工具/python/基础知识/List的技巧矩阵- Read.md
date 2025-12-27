#### 5. ⚡ 极速：索引访问 (Indexing)

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

#### 6. 🔰 基础：负数索引 (Negative Indexing)

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

#### 7. 🐢 较慢：切片读取 (Slicing)

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

#### 8. 🚀 进阶：序列解包 (Unpacking)

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