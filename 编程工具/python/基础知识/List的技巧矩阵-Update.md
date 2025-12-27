#### 1. 🚀 推荐：尾部追加 (Append) —— 唯一的 O(1)

**场景**：向列表添加一个元素。  
**PHP思维**：$arr[] = $val。
```python
data = [1, 2, 3]
new_val = 4

# 🚀 推荐 (Fast/Pythonic) - 摊还 O(1)
data.append(new_val)

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 扩容机制 (Reallocation):
#    - CPython 检查 `ob_size < allocated` ?
#    - Yes: 直接把 `ob_item[ob_size]` 指向新对象，`ob_size++`。极速。
#    - No: 触发扩容。
#      新容量 ≈ 旧容量 + (旧容量 >> 3) + 6。
#      调用 `realloc` 扩展内存块。
#
# 2. 摊还复杂度 (Amortized Analysis):
#    - 虽然偶尔一次 append 会很慢（涉及 realloc），但平均下来，扩容发生的频率很低。
#    - 所以 append 是 List 中**最**推荐的修改操作。
#
# 3. 对比 PHP:
#    - PHP 也会扩容（bucket array doubling），机制类似。
#    - 但 Python 的 append 仅限于**尾部**。
```

#### 2. 💀 性能杀手：头部/中间插入 (Insert)

**场景**：在列表**最前面**加一个元素（比如做队列的 unshift）。  
**PHP思维**：array_unshift($arr, $val)。PHP 是链表，这操作很快（如果是纯链表），但如果是 Packed Array 也会涉及移动，不过 PHP 用户通常无感。
```python
data = [1, 2, 3, 4, 5] # 假设这里有 100万 个元素

# 💀 性能杀手 (Killer) - O(N)
# 动作：插入到索引 0
data.insert(0, 99)

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 内存大挪移 (Memmove):
#    - `insert(0, val)` 意味着：
#      "把索引 0 到 索引 N-1 的所有指针，全部往后挪 1 位，给新元素腾地儿。"
#    - C底层：调用 `memmove(&items[1], &items[0], n * sizeof(PyObject*))`。
#    - 如果 data 有 100万个元素，CPU 就得搬运 8MB 的内存数据。
#    - 这是一个非常重的操作！
#
# 2. 架构权衡 (#面试Tips):
#    - 面试官问：“怎么实现一个队列？”
#    - 绝对不要用 List 的 `insert(0)` 或 `pop(0)`。
#    - **解法**：使用 `collections.deque` (双端队列)。
#      Deque 是双向链表实现的（分块链表），头尾操作都是 O(1)。
```

#### 3. 🚀 推荐：批量扩展 (Extend)

**场景**：把另一个列表的所有元素加到当前列表末尾。  
**PHP思维**：$a = array_merge($a, $b) 或者循环 $a[] = ...。
```python
data = [1, 2]
extras = [3, 4, 5]

# 🚀 推荐 (Fast/Pythonic)
data.extend(extras)
# 或者 data += extras

# 🐢 较慢 (Slow) - 循环 append
# for x in extras:
#     data.append(x)

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 预知未来 (Pre-sizing):
#    - `extend` 知道 `extras` 的长度是 3。
#    - 它只做**一次**扩容检查 (realloc)。
#    - 如果用循环 append，可能会触发多次扩容（比如 2->4->8）。
#
# 2. 批量拷贝:
#    - 扩容后，它直接把 extras 里的指针 `memcpy` 过来。
#    - 循环 append 则是每次都要读写一次 `ob_size` 和指针赋值。
```

#### 4. 💀 性能杀手：切片赋值 (Slice Assignment) 的缩放陷阱

**场景**：替换列表中的一段内容，但新内容长度不同。  
**PHP思维**：array_splice。
```python
nums = [1, 2, 3, 4, 5]

# 💀 性能杀手 (Killer) - 导致扩容和移动
# 动作：把中间 3 个元素替换成 1000 个元素
nums[1:4] = [9] * 1000 

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 复杂的内存手术:
#    - 原切片长度：3 (2,3,4)。
#    - 新内容长度：1000。
#    - 差值：+997。
#    - CPython 必须：
#      1. `realloc` 扩大列表内存。
#      2. 把索引 4 后面的元素 (5) 向后搬运 997 个位置 (memmove)。
#      3. 把那 1000 个新元素的指针拷贝进来 (memcpy)。
#
# 2. 警告：
#    - 虽然语法很酷，但如果在长列表中间这样塞入大量数据，会导致严重的性能抖动。
```

#### 5. 🚀 进阶：修改元素值 (Set Item)

**场景**：data[i] = val。  
**PHP思维**：$arr[$i] = $val。
```python
data = [10, 20, 30]

# ⚡ 极速 (Instant/Atomic) - O(1)
data[1] = 99

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 引用计数更新 (Ref Counting):
#    - 这不仅仅是 `ob_item[1] = new_ptr` 这么简单。
#    - CPython 必须先：
#      1. `old_ptr = ob_item[1]` (拿到原来的 20)。
#      2. `Py_INCREF(new_ptr)` (99 的引用数 +1)。
#      3. `ob_item[1] = new_ptr` (赋值)。
#      4. `Py_DECREF(old_ptr)` (原来的 20 引用数 -1)。
#    - 如果 20 的引用数变成 0，可能会触发 20 这个对象的垃圾回收 (__del__)。
#
# 2. 线程安全 (#面试Tips):
#    - 这个操作在 CPython 中是原子的（因为有 GIL）。
#    - 你不需要担心两个线程同时修改同一个索引会导致指针错乱（但数据逻辑竞争依然存在）。
```

#### 6. 🚀 推荐：列表排序 (Sort)

**场景**：对列表进行原地排序。  
**PHP思维**：sort($arr)。
```python
data = [3, 1, 2]

# 🚀 推荐 (Fast/Pythonic) - O(N log N)
data.sort()

# 🐢 较慢 (Slow) - 创建新列表
# sorted_data = sorted(data)

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. Timsort 算法:
#    - Python 的 `sort` 使用的是 Timsort (归并排序 + 插入排序的混合体)。
#    - 它极其适合处理“部分有序”的真实数据。
#
# 2. 临时内存 (Temporary Memory):
#    - `sort()` 是**原地 (In-place)** 修改。
#    - 它不需要像 `sorted()` 那样创建一个全新的列表对象副本。
#    - 它只移动指针。
#
# 3. Key 函数:
#    - `data.sort(key=lambda x: x.id)`。
#    - 尽量避免复杂的 lambda，这会增加 Python 函数调用开销。
#    - 推荐 `from operator import itemgetter`，它是 C 实现的，比 lambda 快。
```