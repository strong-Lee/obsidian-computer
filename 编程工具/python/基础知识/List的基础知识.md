## 1. 基础常用

```python
# 1. 快速构建-内存初始化：Python 会申请一块能装100个指针的内存，并将它们都指向同一个整数对象 0 的地址（引用拷贝）。极快，O(n)。
nums = [0] * 100

# 2. 负数索引-语法糖：CPython 内部处理逻辑为 index = len + index。如果相加后仍为负，则抛出 IndexError。O(1)。
l = ["a", "b", "c"] 
print(l[-1]) # 等价于 l[len(l) - 1] -> l[2]

# 3. 判空- PEP8规范：直接检查 C 结构体中的 ob_size 是否为 0，比 if len(nums) == 0 少了一次函数调用。
if not nums:
	print("Empty")
	
# 4. 插入-搬运代价：这在 Python 中是`大忌`。因为内存连续，插入头部意味着后面 N 个元素都要向后 memmove（移动）。O(n)。对比 PHP 链表插入是 O(1)。
nums.insert(0, val)

# 3. 切片复制-深浅拷贝：申请新内存块，将原列表的`对象指针`复制过去。不复制对象本身。O(n)。如果列表中包含可变对象（如子列表），修改子列表会影响原列表。必须用 copy.deepcopy。
copy = nums[:]
import copy 
l1 = [[1], 2] 
l2 = l1[:] # 浅拷贝 
l3 = copy.deepcopy(l1) # 深拷贝
l2[0][0] = 999 
print(l1) # [[999], 2] -> 原列表被改了！ 
print(l3) # [[1], 2] -> 深拷贝不受影响

# 4. 步长切片-反转：创建一个新列表，按步长 -1 读取原列表指针并填入。O(n)。
rev = nums[::-1]


	
# 6. 拼接-新建：malloc 一块大小为 len(a)+len(b) 的新内存，先 memcpy a，再 memcpy b。O(m+n)。
c = a + b
# 7. 原地追加-扩容：在 a 的内存基础上 realloc，减少了一次内存分配和旧数据复制，比 a = a + b 高效得多。
a.extend(b) #或 a += b

# 9. 弹出-尾部操作：仅仅是将 ob_size 减 1，不需要移动内存。O(1) 且极快。
val = nums.pop()
# 10. 排序-Timsort：Python 的默认排序算法，稳定且对部分有序数据极快。原地修改，不费额外内存。
nums.sort()
# 11. 存在性检查-遍历：线性查找，O(n)。不像 PHP 检查 Key 是 O(1)，这里必须挨个比对。
if x in nums:
# 12. 清空释放：将内部指针数组引用计数减 1，ob_size 设为 0，但可能保留已分配的内存容量（allocated）以便复用。
nums.clear()
```
## 2. 进阶高级

```python
# 1. 推导式: 字节码优化，比 for 循环快，因为是在 C 语言层面循环构建列表，减少了 Python 虚拟机栈帧的开销。
[x*2 for x in nums]
[x for x in nums if x > 0]    # 结合了 filter 和 map 功能，高度优化的构建过程。
[x**2 for x in range(10) if x % 2 == 0]
# 2. 解包：协议，利用迭代器协议，自动切分列表。底层进行了多次切片操作。
first, *mid, last = nums
# 3. 带索引遍历-代器：生成一个元组流 (index, value)，避免了手动维护计数器 i += 1。
for i, v in enumerate(nums):
# 4. 并行迭代-惰性求值：zip返回迭代器，按需从两个列表中取值，内存友好的O(1)空间复杂度。
for a, b in zip(list1, list2):
# 5. 列表扁平化-高频操作：虽然能把二维数组铺平，但会产生大量临时中间列表，产生 O(n²) 复杂度。慎用。
for a, b in zip(list1, list2):
# 6. 自定义排序键：利用 key 函数预处理比较键值，避免在比较时重复计算。
nums.sort(key=lambda x: x['age'])
lst.sort(key=lambda x: len(str(x))) 
# 7. 列表转字符串-一次性分配：计算所有字符串总长度，一次 malloc，然后逐个复制。绝对不要用 += 拼接字符串。
''.join(str_list)
# 8. 生成器表达式-短路求值：不会生成完整列表，找到满足条件的就立刻停止并返回 True，节省内存和时间。
any(x > 10 for x in nums)
# 9. 删除切片-批量删除：支持步长删除，CPython 会计算保留元素的偏移量并进行批量内存移动。
del nums[::2]
# 10. 哨兵值-Hack：持续 pop 直到遇到哨兵值，用于消费队列直到空。
iter(list_obj.pop, sentinel)
# 11. 也就是假-引用陷阱：创建了包含三个指向同一个空列表指针的列表。改一个全变。
l = [[]] * 3
# 12. 双端队列-数据结构选择：列表的头部插入/删除是 O(N) 的（因为要移动后面所有数据）。如果需要频繁在头部操作（如队列），请务必使用 deque，它是 O(1) 的。
from collections import deque
q = deque([1, 2, 3])
q.popleft() # O(1) 极快
# vs
l = [1, 2, 3]
l.pop(0)    # O(N) 很慢，慎用
```
## 3. 黑客技巧

```python

# 1. 预分配-避免扩容：如果你知道长度，先占位。避免 append 触发多次 realloc（扩容）。
l = [None] * N
nums = []; for i in range(10000): nums.append(i); # 慢，触发多次扩容
nums = [None] * 10000; for i in range(10000): nums[i] = i; # 快，一次性分配

# 2. 引用计数-GC：查看有多少变量引用这个列表。这是python垃圾回收（GC）的基础。注意调用该函数时，参数本身作为临时引用，计数通常会比预想多 1。
# sys.getrefcount(nums)
import sys 
a = [1, 2] 
b = a 
print(sys.getrefcount(a)) # 输出 3 (a, b, 加上 getrefcount 的参数)

# 3. 浅层大小-元数据：只计算 List 结构体 + 指针数组的内存占用，不包含元素对象本身的大小。列表存的只是“引用的指针”（通常 8 字节/个）。
sys.getsizeof(nums)
import sys 
l = ["a" * 1000] # 列表本身很小（只存了指针），但实际字符串占用内存很大 
print(sys.getsizeof(l)) # ~72 bytes (列表壳子)

# 4. 内存地址-指针验证：返回对象的内存地址。常用于判断两个变量是否指向同一个对象（即 is 判断的原理）。
id(nums[0])
a = [1, 2] 
b = a 
print(id(a) == id(b)) # True，说明是同一个对象 
print(a is b) # True

# 5. 全量替换-原地更新：极重要技巧。保持列表对象的内存地址（ID）不变，但替换其所有内容。常用于函数内部修改外部列表（如 LeetCode 题目），应用场景：多线程或多模块共享一个列表对象作为缓冲区，不希望改变对象引用的情况下更新数据。
nums[:] = new_nums
def modify(nums): 
	# nums = [1, 2] # 错误：这只是让 nums 变量指向了新列表，外部变量不受影响
	nums[:] = [1, 2] # 正确：在原内存地址上修改数据 
a = [0, 0] 
modify(a) 
print(a) # [1, 2]

# 6. 扩容策略-分配策略：Python 列表使用了超额分配（Over-allocation）策略来保证O(1)的追加性能。扩容系数约为 1.125 倍（加上少量常数）。
# 观察 大小的跳跃：32-> 64 -> 96...(字节数视 Python 版本而定)
import sys 
l = [] 
for i in range(5): 
	l.append(i) 
	print(len(l), sys.getsizeof(l))

# 7. 线程安全-GIL机制：CPython 中，单一字节码操作（如 append）是原子的，线程安全的。但 l[i] = l[j] + 1 涉及读写两步，非线程安全不是。
# 例子：你有一个全局列表 logs = []，10个线程同时往里面塞日志。
# - 安全：使用 logs.append(data)。因为 CPython 的 GIL 保证这一步不会被打断，数据不会丢失。
# - 不安全：如果做 logs[0] += 1（统计计数）。这分为三步：取值、加1、赋值。线程 A 取了值还没赋值，线程 B 也取了旧值。结果两个线程加完，只增加了 1。这就是数据竞争，需要加锁。
append, pop
L.append(x)   # 安全 
L[0] += 1     # 不安全 (多线程下需要 Lock) 

# 8. 小整数缓存-对象池：范围 -5 到 256 的整数是全局唯一单例。List 存的是这些全局单例的指针，不会重复创建对象，节省内存。
l = [1, 2, 3] (Small Int)
l = [1, 1, 1] # 三个元素指向同一个内存地址 
print(id(l[0]) == id(l[1])) # True

# 9. 紧凑型数组-去指针化：如果列表只存数字，使用 array 模块存储 C 语言原生的 int/float，而非 PyObject 指针。内存占用可减少 2/3 以上。且对 CPU 缓存友好。
import array
array.array('i', [1,2]) # 'i' 代表 signed int

# 10. 避免多态开销-性能陷阱：List 可存任意类型，导致 CPU 无法进行向量化优化，且每次读取都要检查类型。处理大量同质数据（如矩阵运算），请务必使用 NumPy。
# Python List: 慢，逐个对象解包检查类型 
l = [1, 2.0, "3"] 
# NumPy Array: 快，内存连续，类型统一 
import numpy as np 
arr = np.array([1, 2, 3], dtype=int)
# 如果不做矩阵运算只为了省内存，可以用array模块

# 11. 自定义排序-兼容旧代码：将老式的 cmp(a, b) 函数转换为 key 函数，利用了类包装器。如果需要复杂的比较逻辑（如：A 比 B 大返回 1），可用 functools.cmp_to_key 将比较函数转为 key 函数。
from functools import cmp_to_key
# 自定义比较：按绝对值大小倒序
def my_cmp(a, b):
    return abs(b) - abs(a)

nums = [1, -5, 3]
nums.sort(key=cmp_to_key(my_cmp)) # [-5, 3, 1]

# 12. 列表去重-保留顺序：利用 Python 3.7+ 字典有序的特性去重，比 list(set(nums)) (无序) 更稳健，保留插入顺序。
nums = [3, 1, 2, 1, 3]
unique = list(dict.fromkeys(nums)) # [3, 1, 2]
unique_set = list(set(nums))       # [1, 2, 3] (顺序不确定)

# 13. 矩阵转置-骚操作：利用解包操作符*和zip函数，可以一行代码实现矩阵行列互换（转置）。
matrix = [[1, 2, 3], 
          [4, 5, 6]]
transposed = list(zip(*matrix)) # 结果: [(1, 4), (2, 5), (3, 6)]

# 14. 二分查找维护-算法优化：在一个**已排序**的列表中插入元素并**保持排序**，千万别用 append() + sort()（O(N log N)）。使用 bisect.insort 可以 O(N) 完成。
import bisect
scores = [60, 70, 80, 90]
# 查找 75 应该插入的位置，并插入
bisect.insort(scores, 75) 
print(scores) # [60, 70, 75, 80, 90]
```