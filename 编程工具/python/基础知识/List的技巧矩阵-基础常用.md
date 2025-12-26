
```python
1. 创建列表： 看似简单，实则分配了定长指针数组
# 动作：CPython 调用 PyList_New(3)。 
# 原理：分配 PyListObject 结构体 + 3个 PyObject* 指针的连续空间。 
# 复杂度：O(N)。
# PHP： $a = []; (底层初始化哈希表)
data = [1, 2, 3]
# 快速构建-内存初始化：Python 会申请一块能装100个指针的内存，并将它们都指向同一个整数对象 0 的地址（引用拷贝）。极快，O(n)。
data = [0] * 100

2. 尾部追加： 最推荐的操作# 动作：检查 allocated > ob_size? 是：直接填入。否：realloc 扩容。 
# 原理：Amortized O(1) (均摊O(1))。虽然偶尔扩容慢，但平均极快。
# PHP：$a[] = 4;
data.append(4)

3. 指定位置插入-性能杀手：这在 Python 中是`大忌`。因为内存连续，插入头部意味着后面 N 个元素都要向后 memmove（移动）。O(n)。对比 PHP 链表插入是 O(1)。
# 动作：将 index 0 之后的所有元素向后挪动一位（memmove）。 
# 原理：这是连续内存！就像在排队的人群最前面插队，后面所有人都要后退一步。 
# 复杂度：O(N)。这就是 List 不适合做先进先出队列的原因（请用 collections.deque）。
# PHP: array_splice($a, 0, 0, 999); 
data.insert(0, val)
# 极快 O(1) - 如果需要频繁头插，请用 deque, 是一个双向链表，但每个节点存的不是一个元素，而是一个由指针组成的数组（Block）。C语言类比：
# struct block { 
#	 PyObject *items[64]; 
#    struct block *left; 
#    struct block *right; 
# }。 
from collections import deque 
q = deque(nums); 
q.appendleft(val)

4. 判断元素存在-遍历：list是线性查找，O(n)。不像 PHP 检查 Key 是 O(1)，这里必须挨个比对。Set/Dict 是哈希查找 O(1)。数据量大时，请转为 set 再查。
# 动作：从头遍历到尾，调用 PyObject_RichCompareBool 逐个比较。 
# 原理：没有哈希查找！ 
# 复杂度：O(N)。数据量大时，请转 set (O(1))。
# PHP: in_array(2, $a);
if x in nums: # O(N) 慢
if x in set(nums): # O(1) 快（但转换本身有开销）

5. 长度获取：极速
# 动作：直接读取结构体字段 ob_size。 
# 原理：PyListObject 内部维护了当前长度，无需遍历。 
# 复杂度：O(1)。
# PHP: count($a); 
n = len(data)

6. 尾部弹出-极速：仅仅是将 ob_size 减 1，被删除元素的指针会被清除（引用计数-1），不需要移动内存。O(1) 且极快。
# 动作：ob_size 减 1。原来的指针被“遗忘”（引用计数减1）。 
# 原理：无需内存收缩（shrink），保留空间给未来 append。 
# 复杂度：O(1)。
# PHP: array_pop($a); 
last = data.pop() # 默认弹出最后一个

7. 弹出头部：性能杀手。
# 动作：移除第0个，后面所有元素向前挪动一位 (memmove)。 
# 原理：填补空缺。 
# 复杂度：O(N)。 
# PHP: array_shift($a); 
first = data.pop(0) 

8. 清空列表-资源释放：将内部指针数组引用计数减 1，ob_size 设为 0，但不一定释放内存容量（allocated），以便下次 append 时复用内存。
# 动作：ob_size 设为 0，释放所有元素的引用计数，但可能保留allocated内存块以便复用。 
# 复杂度：O(N) (因为要处理每个元素的引用计数)。
data.clear() 

9. 列表反转：原地修改。 
# PHP: $a = array_reverse($a); (返回新数组) 
# 动作：收尾对调。 
# 原理：不申请新内存，直接交换指针。 
# 复杂度：O(N)。
data.reverse() 

10. 排序-Timsort：Python 的默认排序算法，稳定且对部分有序数据极快。原地修改，不费额外内存。
# PHP: sort($a); (快排/归并) 
# 动作：使用 Timsort（结合归并和插入排序）。 
# 原理：Python 的骄傲。利用数据的“天然有序性”进行优化。 
# 复杂度：O(N log N)。
data.sort() 


# 负数索引-语法糖：CPython 内部处理逻辑为 index = len + index。如果相加后仍为负，则抛出 IndexError。O(1)。
l = ["a", "b", "c"] 
print(l[-1]) # 等价于 l[len(l) - 1] -> l[2]

# 判空- PEP8规范：直接检查 C 结构体中的 ob_size 是否为 0，比 if len(nums) == 0 少了一次函数调用。
if not nums:
	print("Empty")
	

# 拼接陷阱-内存分配：
# a + b：新建。Malloc 一块 len(a)+len(b) 的新内存，复制 a，再复制 b。O(M+N)。
# a += b (即 extend)：扩容。在 a 的原内存上 realloc（如果够大甚至不用动），直接把 b 的数据拷在后面。省去了一次大内存分配和 a 的复制。
c = a + b  # 慢，产生垃圾对象
a += b     # 快，原地修改, 等价于a.extend(b) 操作：它检查 a 屁股后面有没有空闲内存。有：直接把b的数据拷贝进去。没有：申请一块更大的新地盘，把a搬过去，再把b拷进去。

# 引用陷阱-经典bug：创建了包含三个指向同一个空列表指针的列表。改一个全变。
l = [[]] * 3
l[0].append(1) 
print(l) # [[1], [1], [1]] -> 灾难现场 
l = [[] for _ in range(3)] # 正确写法 


```
