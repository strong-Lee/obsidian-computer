#### 1. 🔰 基础：字面量 vs 构造函数 (Literal vs Constructor)

**场景**：创建一个空列表。
```python
# 🚀 推荐 (Fast/Pythonic)
empty_list = []

# 🐢 较慢 (Slow)
# 动作：调用 list() 类构造函数
slow_list = list()

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. [] (BUILD_LIST Opcode):
#    - Python 解释器直接识别为字节码指令 `BUILD_LIST`。
#    - C底层：直接调用 `PyList_New(0)`，分配 `PyListObject` 结构体。
#    - ⚡ 极速：没有函数调用栈的开销。
#
# 2. list() (Function Call):
#    - 这是一个函数调用！
#    - 步骤 A：去 global 命名空间找 'list' 这个名字。
#    - 步骤 B：去 builtin 命名空间找 'list' 类。
#    - 步骤 C：压栈、执行构造函数、弹栈。
#    - 🐢 较慢：比 [] 慢约 2-3 倍（微秒级差异，但在高频循环中致命）。
#
# 3. 对比 PHP：
#    - PHP 的 $a = array() 和 $a = [] 几乎等价。
#    - 但 Python 的 list() 是真函数，[] 是指令。
```

#### 2. 🏴‍☠️ 黑客：预分配内存 (Pre-allocation)

**场景**：你需要生成一个包含 100 万个元素的列表（比如全 0）。  
**PHP思维**：$arr = []; for($i=0...){ $arr[] = 0; } (PHP 也会自动扩容，但我们通常不关心)。
```python
N = 1000000

# 💀 性能杀手 (Killer) - 类似 PHP 的写法
# data = []
# for i in range(N):
#     data.append(0)  # 触发多次 realloc 和 数据搬运

# 🚀 推荐 (Fast/Pythonic)
# 动作：利用列表乘法进行预分配
data = [0] * N

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 内存动作 (C Malloc):
#    - Python 知道最终大小是 N。
#    - C底层：一次性 `malloc(N * sizeof(PyObject*))`。
#    - 避免了 `append` 过程中的多次 `realloc` (内存重分配) 和 `memmove` (数据旧址搬运到新址)。
#
# 2. 引用陷阱 (Ref Trap):
#    - data = [0] * N 实际上是复制了 N 个指向同一个整数对象 `0` 的指针。
#    - 因为 `int` 是不可变的 (Immutable)，所以这很安全。
#    - ⚠️ 警告：如果是 data = [[]] * N (列表的列表)，修改其中一个，所有的都会变！(浅拷贝指针)。
#
# 3. 面试话术 (#面试Tips):
#    - 问：“Python 列表是如何扩容的？”
#    - 答：“它是指数级超额分配 (Over-allocation)。比如存第 1 个元素，它可能给 4 个位置；
#          存第 5 个，给 8 个... 增长系数约为 1.125 (Python 3.9+)。
#          使用 `[None] * N` 可以一步到位，避免扩容抖动。”
```

####   3. 🚀 进阶：列表推导式 (List Comprehension)

**场景**：将一个列表的数字平方。  
**PHP思维**：array_map 或者 foreach 循环。
```python
nums = [1, 2, 3, 4, 5]

# 🚀 推荐 (Fast/Pythonic)
squares = [x**2 for x in nums]
[x**2 for x in range(10) if x % 2 == 0]

# 🐢 较慢 (Slow) - 手动循环
# res = []
# for x in nums:
#     res.append(x**2)

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 字节码差异 (Bytecode):
#    - 循环版：每次循环都要执行 `LOAD_ATTR (append)` 和 `CALL_FUNCTION`。
#    - 推导式版：C 语言层面的 `LIST_APPEND` 指令。
#    - 这里的关键是：推导式是在 C 语言的循环里跑，而 `for` 循环是在 Python 虚拟机的循环里跑。
#
# 2. 速度对比：
#    - 推导式通常比手写 for 循环快 30% ~ 50%。
#
# 3. 对比 PHP：
#    - PHP 的 array_map 也是 C 实现的，速度很快。
#    - Python 的 map() 函数也很快，但返回的是迭代器，转 list 还要消耗。推导式是最佳平衡点。
```

#### 4. 🏴‍☠️ 黑客：深浅拷贝的指针游戏

**场景**：复制一个列表。  
**PHP思维**：PHP 默认是 **Copy-on-write (写时复制)**。$b = $a 看起来是复制，实际是共享内存，直到你修改 $b，PHP 才会真正复制内存。  
**Python思维**：Python 默认是 **Reference (引用)**。
```python
row = [1, 2, 3]
matrix = [row, row] # [[1,2,3], [1,2,3]]

# 🚀 推荐 (Fast/Pythonic) - 浅拷贝 (Shallow Copy)
copy_matrix = matrix[:] 
# 或者 copy_matrix = list(matrix)
# 或者 import copy; copy.copy(matrix)

# 🐢 较慢 (Slow) - 深拷贝 (Deep Copy)
import copy
deep_matrix = copy.deepcopy(matrix)

# 💡 源码级剖析 (Source Code Analysis)
# --------------------------------------------------------------------
# 1. 赋值 (=):
#    - `b = matrix` 仅仅是多了一个标签指向同一个 `PyListObject`。指针完全一致。
#
# 2. 切片 [:] (Shallow Copy):
#    - 动作：创建一个新的 `PyListObject`，申请新的内存块。
#    - 内存：将原列表中存储的 `PyObject*` 指针，用 `memcpy` 复制到新列表。
#    - 引用计数 (Ref Count)：列表中每个元素的 `ob_refcnt` +1。
#    - 坑：虽然列表壳子是新的，但里面的元素(row) 还是指向原来的内存地址！
#      修改 copy_matrix[0][0] = 99，原 matrix 也会变！因为它们指着同一个 row。
#
# 3. PHP 对比：
#    - PHP 的 $b = $a 让你觉得很安全。
#    - Python 必须时刻警惕：我在操作指针，还是在操作对象？
```
