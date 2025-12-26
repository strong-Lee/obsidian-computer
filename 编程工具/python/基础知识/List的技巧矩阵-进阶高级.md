
```python
1. 列表推导式-字节码优化：
# 比 for 循环快，因为推导式在编译层级有专门的指令（LIST_APPEND），是在 C 语言层面循环，减少了 Python 虚拟机栈帧压栈/出栈的开销。
# 动作：C 语言层面直接构建列表，减少了 LOAD_ATTR (append) 的字节码查找开销。 
# 原理：LIST_APPEND 字节码指令比方法调用更底层。 
# 复杂度：O(N)。
# 慢写法：res = []; for i in range(10): res.append(i*2) 
res = [i * 2 for i in range(10)] 
[x*2 for x in nums]
[x for x in nums if x > 0]    # 结合了 filter 和 map 功能，高度优化的构建过程。
[x**2 for x in range(10) if x % 2 == 0]

2. 切片复制-深浅拷贝：
# 申请新内存块，将原列表的`对象指针`复制过去。不复制对象本身。O(n)。如果列表中包含可变对象（如子列表），修改子列表会影响原列表。必须用 copy.deepcopy。
# 动作：申请新 PyListObject，malloc 新指针数组，将原列表对应范围的 指针 复制过去。 
# 原理：引用计数增加，但不复制对象本身！ 
# 复杂度：O(K)，K为切片长度。
sub = data[1:3] # 浅拷贝
import copy 
l1 = [[1], 2] 
l2 = l1[:] # 浅拷贝 
l3 = copy.deepcopy(l1) # 深拷贝
l2[0][0] = 999 
print(l1) # [[999], 2] -> 原列表被改了！ 
print(l3) # [[1], 2] -> 深拷贝不受影响

3. 步长切片-反转技巧：
# 创建一个新列表，按步长 -1 读取原列表指针并填入。O(n)。这是生成新列表，而 nums.reverse() 是原地反转。
# 动作：按步长计算索引，复制指针。 
# 复杂度：O(N/Step)。
evens = data[::2] 
s = "hello" 
rev = s[::-1] # "olleh"

4. 切片删除-批量操作：
# 支持步长删除。CPython 会计算保留元素的偏移量，直接进行批量的内存移动（memmove），比循环 pop 快得多。
# 动作：计算保留的元素，搬运到紧凑位置，调整 ob_size。 
# 复杂度：O(N)。
nums = [1, 2, 3, 4, 5, 6]
del nums[::2] # 删除索引 0, 2, 4 -> 结果 [2, 4, 6]

5. 解包协议-星号解包：
# Python 3 的特性。利用迭代器协议，自动切分列表。底层进行了多次切片操作，代码极简。
# 动作：head 取 data[0]，tail 创建新列表包含 data[1:]。 
# 原理：Python 的迭代协议
# PHP: list($a, $b) = $arr;
nums = [1, 2, 3, 4, 5]
first, *mid, last = nums
# first=1, mid=[2,3,4], last=5

6. 带索引遍历-代器：
# 生成一个元组流 (index, value)，避免了手动维护计数器 i += 1。
# 动作：生成器 yield (index, value) 元组。 
# 原理：避免了自己在外部维护 index 变量。
# PHP: foreach ($data as $i => $val)
for i, v in enumerate(data):
	print(i, v)

7. 并行迭代-惰性求值：
# zip返回迭代器，按需从两个列表中取值，内存友好的O(1)空间复杂度。
# 动作：同时迭代两个列表，最短截断。 
# 原理：不需要像 PHP 那样通过索引 $names[$i], $ages[$i] 访问。
names = ['A', 'B']; scores = [90, 80]
for n, s in zip(names, scores):
    print(n, s)
    
8. 短路求值-生成器表达式：
# any(x > 10 for x in nums) 不会生成完整列表。它像雷达一样，一旦扫描到满足条件的元素，立刻停止并返回 True。节省内存和时间。
# 动作：一旦发现满足条件的，立即停止迭代。 
# 原理：类似 && 和 || 的短路特性，但在集合层面。
has_large = any(x > 1000 for x in range(1000000)) # 瞬间返回
if any(x > 10 for x in data): 
	pass
        
# 扁平化-性能对比：sum 是反面教材 O(N²)。itertools.chain 是标准答案，零内存拷贝，O(N)。
# zip：是拉链。把 [1,2]和[3,4]的第一位扣在一起，第二位扣在一起。用于并行迭代或转置。
# chain：是锁链。把 [3,4] 接在 [1,2] 后面。用于扁平化。
# sum(l, [])：是灾难。它相当于 (list1 + list2) + list3。每加一次，都要创建新列表并复制之前所有数据。O(N²)。
# chain 的原理是迭代器，它不创建新列表，只是像指针一样，“读完A表指B表”，内存 O(1)，速度极快。
flat = sum([[1], [2], [3]], [])  # 极慢：1+2, (1+2)+3, (1+2+3)+4...

matrix = [[1, 2], [3, 4], [5, 6]]
flat = sum(matrix, []) # 极慢
# 极快 (返回迭代器，转 list 需强转) 
import itertools 
flat_iter = itertools.chain.from_iterable(matrix) 
flat_list = list(flat_iter)


# 字符串拼接-一次性分配：绝对不要用 += 拼接字符串。join 会先计算所有字符串的总长度，一次性 malloc 内存，然后逐个 memcpy。效率是天壤之别。
# 每次 += 都创建新字符串对象 
s = ""; 
for x in l: s += x 
# 高效 
s = "".join(l)




# 自定义排序键：利用 key 函数预处理比较键值，避免在比较时重复计算。
nums.sort(key=lambda x: x['age'])
lst.sort(key=lambda x: len(str(x))) 

# 哨兵值-Hack：持续 pop 直到遇到哨兵值，用于消费队列直到空。
iter(list_obj.pop, sentinel)

# 双端队列-数据结构选择：列表的头部插入/删除是 O(N) 的（因为要移动后面所有数据）。如果需要频繁在头部操作（如队列），请务必使用 deque，它是 O(1) 的。
from collections import deque
q = deque([1, 2, 3])
q.popleft() # O(1) 极快
# vs
l = [1, 2, 3]
l.pop(0)    # O(N) 很慢，慎用

# 词频统计-神器：底层是 C 实现的哈希计数，比手写 dict 循环快且优雅。
from collections import Counter nums = [1, 1, 2, 3, 1] counts = Counter(nums) # Counter({1: 3, 2: 1, 3: 1}) 
top_k = counts.most_common(2) # 获取出现频率最高的2个
```
