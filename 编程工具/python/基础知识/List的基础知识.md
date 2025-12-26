## 1. 基础常用

```python
# 1. 快速构建：内存初始化，Python 会申请一块能装100个指针的内存，并将它们都指向同一个整数对象 0 的地址（引用拷贝）。O(n)。
nums = [0] * 100
# 2. 负数索引：语法糖：CPython 内部处理逻辑为 index = len + index。O(1)。
last = nums[-1]
# 3. 切片复制：浅拷贝，申请新内存块，将原列表的`对象指针`复制过去。不复制对象本身。O(n)。
copy = nums[:]
# 4. 步长切片：反转，创建一个新列表，按步长 -1 读取原列表指针并填入。O(n)。
rev = nums[::-1]
# 5. 判空：性能，直接检查 C 结构体中的 ob_size 是否为 0，比 len(nums) == 0 更快
if not nums:
# 6. 拼接：新建，malloc 一块大小为 len(a)+len(b) 的新内存，先 memcpy a，再 memcpy b。O(m+n)。
c = a + b
# 7. 原地追加：扩容，在 a 的内存基础上 realloc，减少了一次内存分配和旧数据复制，比 a = a + b 高效得多。
a.extend(b) #或 a += b
# 8. 插入：搬运代价，这在 Python 中是`大忌`。因为内存连续，插入头部意味着后面 N 个元素都要向后 memmove（移动）。O(n)。对比 PHP 链表插入是 O(1)。
nums.insert(0, val)
# 9. 弹出：尾部操作，仅仅是将 ob_size 减 1，不需要移动内存。O(1) 且极快。
val = nums.pop()
# 10. 排序：Timsort，Python 的默认排序算法，稳定且对部分有序数据极快。原地修改，不费额外内存。
nums.sort()
# 11. 存在性检查：遍历，线性查找，O(n)。不像 PHP 检查 Key 是 O(1)，这里必须挨个比对。
if x in nums:
# 12. 清空：释放，将内部指针数组引用计数减 1，ob_size 设为 0，但可能保留已分配的内存容量（allocated）以便复用。
nums.clear()
```

## 2. 进阶高级

```python
# 13. 推导式: 字节码优化，比 for 循环快，因为是在 C 语言层面循环构建列表，减少了 Python 虚拟机栈帧的开销。
[x*2 for x in nums]
[x for x in nums if x > 0]    # 结合了 filter 和 map 功能，高度优化的构建过程。
[x**2 for x in range(10) if x % 2 == 0]
# 14. 解包：协议，利用迭代器协议，自动切分列表。底层进行了多次切片操作。
first, *mid, last = nums
# 15. 带索引遍历-代器：生成一个元组流 (index, value)，避免了手动维护计数器 i += 1。
for i, v in enumerate(nums):
# 16. 并行迭代-惰性求值：zip返回迭代器，按需从两个列表中取值，内存友好的O(1)空间复杂度。
for a, b in zip(list1, list2):
# 17. 二维转一维-反模式：虽然能把二维数组铺平，但会产生大量临时中间列表，产生 O(n²) 复杂度。慎用。
for a, b in zip(list1, list2):
```


1. **带索引遍历：** for i, v in enumerate(lst):。


3. **自定义排序键：** lst.sort(key=lambda x: len(str(x))) 按长度排序。
4. **原地切片赋值：** lst[1:3] = [10, 20, 30] 替换并改变长度。
5. **filter与map：** 配合 lambda 进行函数式编程。
6. **any() 和 all()：** if any(x > 10 for x in lst): 检查条件。
7. **二维列表转一维：** sum(nested_list, []) (仅限小列表，慢) 或 [item for sublist in nested in sublist]。
8. **bisect 模块：** 在有序列表中进行二分查找插入 bisect.insort(lst, x)。

## 3. 黑客技巧

[](https://github.com/strong-Lee/obsidian-computer/blob/main/%E7%BC%96%E7%A8%8B%E5%B7%A5%E5%85%B7/python/%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/list.md#3-%E9%BB%91%E5%AE%A2%E6%8A%80%E5%B7%A7)

1. **原地清空：** lst[:] = [] 或 lst *= 0 (比 clear() 更底层，保持对象引用不变)。
2. **引用陷阱利用：** row = [0] * 3; matrix = [row] * 3 (修改一个元素会变整列，虽然通常是bug，但有时用于同步状态)。
3. **利用 sys.getsizeof：** 列表预分配内存策略是“超额分配”，观察内存增长规律。
4. **快速转置矩阵：** list(zip(*matrix))。
5. **无限迭代器转列表：** 利用 iter 的哨兵模式 list(iter(func, sentinel))。
6. **getitem 魔法：** 让自定义类像列表一样支持切片。
7. **线程安全性：** append 和 pop 操作在 CPython 中是原子的（线程安全），但 i += 1 不是。
8. **列表作为默认参数的坑：** def f(l=[]): (用于实现函数级缓存)。
9. **深拷贝加速：** 在某些特定结构下，序列化再反序列化（pickle/json）可能比 deepcopy 快。
10. **删除遍历中的元素：** 倒序遍历 for i in range(len(lst)-1, -1, -1) 删除元素，避免索引错位。