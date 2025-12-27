#### [Tips 1] 不要继承 dict (Subclassing Dict)

**场景**：你想创建一个自定义字典，比如“自动记录访问日志的字典”。
**PHP思维**：继承 ArrayObject。

```python
# 💀 性能杀手 (Killer) / 行为不一致
class MyDict(dict):
    def __setitem__(self, key, value):
        print(f"Setting {key}")
        super().__setitem__(key, value)

d = MyDict()
d["a"] = 1 # 打印 "Setting a" -> 正常
d.update({"b": 2}) # ❌ 不打印！直接调用了 C 层的 update，绕过了你的 Python __setitem__

# 🚀 推荐 (Fast/Pythonic) - 继承 UserDict
from collections import UserDict

class MyUserDict(UserDict):
    def __setitem__(self, key, value):
        print(f"Setting {key}")
        super().__setitem__(key, value)
    
    # UserDict 的 update 方法是用 Python 写的一个循环，它会老实调用 __setitem__

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：方法重写失效。
# 2. 底层原理 (C API Shortcuts):
#    - 内置类型 (`dict`, `list`) 的方法（如 `update`）通常为了性能，直接调用 C 语言层面的函数指针 (`PyDict_Merge`)。
#    - 这些 C 函数往往**不查找**也不调用你在 Python 层面重写的方法（如 `__setitem__`）。
#    - `UserDict` 是一个纯 Python 的 Wrapper，它持有 `self.data` (真实字典)，强制所有操作都走 Python 方法路由。
# 3. 权衡：
#    - `dict` 子类：快，但行为不一致。
#    - `UserDict`：慢（多一层封装），但行为完全符合预期。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：为什么继承 dict 有坑？
# 策略：“因为 CPython 为了优化速度，内置方法的 C 实现往往会忽略子类的 Python 重写。要做自定义容器，请继承 `collections.UserDict`。”
```

#### [Tips 2] 可变默认参数 (Mutable Default Argument) —— 经典面试题的底层解

**场景**：函数参数默认是一个空字典。
**PHP思维**：PHP 的默认值是编译时常量或字面量，每次调用都是新的。

```python
# 💀 逻辑炸弹 (Logic Bomb)
def add_user(user, cache={}): 
    cache[user] = True
    return cache

print(add_user("u1")) # {'u1': True}
print(add_user("u2")) # {'u1': True, 'u2': True} -> 之前的 cache 被保留了！

# 🚀 推荐 (Fast/Pythonic)
def add_user_safe(user, cache=None):
    if cache is None:
        cache = {} # 运行时创建新对象
    cache[user] = True
    return cache

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：函数对象初始化。
# 2. 底层原理 (Function Object Creation):
#    - `def` 语句在 Python 中是**可执行代码**。当模块被加载时，`def` 执行，创建函数对象。
#    - 默认参数值 (`{}`) 在此时被计算，并保存在函数对象的 `__defaults__` 属性中（这是一个 Tuple）。
#    - 每次调用函数，如果不传参，就直接从 `__defaults__` 里取出那个**同一个**字典对象的指针。
# 3. 内存动作：
#    - 全局共享同一个堆内存对象。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：为什么默认参数不能是字典？
# 策略：“因为函数定义只执行一次，默认参数是静态绑定的。对于 Mutable 对象，这意味着所有调用共享同一个实例。这也是实现‘静态变量’的一种 Hack 方式（虽然不推荐）。”
```
#### [Tips 3] 哈希值 -1 的陷阱 (The Hash -1 Trap)

**场景**：你自己写 C 扩展或者深度 Debug 时。
🏴‍☠️ 黑客/底层 (Hacker/Internals)

```python
# 普通情况
hash(1) # -> 1
hash(2) # -> 2

# 诡异情况
hash(-1) # -> -2 
# 等等，为什么不是 -1？
hash(-2) # -> -2

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：Python 的整数 Hash 通常是其本身。
# 2. 底层原理 (CPython Error Code):
#    - 在 CPython 的 C API 中，`PyObject_Hash` 函数返回 `long`。
#    - **-1 被保留作为错误代码**（表示发生了 Exception）。
#    - 因此，如果一个对象的真实 Hash 计算结果是 -1，CPython 会强制将其改为 -2。
# 3. 影响：
#    - 这导致整数 `-1` 和 `-2` 的 Hash 值碰撞了！
#    - 虽然字典能处理冲突（通过比较值相等性），但这是底层的一个微小性能损耗点。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：hash(-1) 等于多少？
# 策略：“等于 -2。这是因为 -1 在 CPython C API 中是错误码。这是 Python 实现细节中的一个妥协。”
```

#### [Tips 4] 字典大小变化限制 (Size Mutation Limit)

**场景**：在多线程或异步代码中，依赖 len(d)。
🏴‍☠️ 黑客/底层 (Hacker/Internals)

```python
d = {i: i for i in range(100)}

# 假设另一个线程正在疯狂 del d[k]

# 💀 逻辑炸弹
# if len(d) > 0:
#    k = d.keys()[0] # 可能会 IndexError！
# 因为在 len(d) 和 keys() 之间，字典可能空了。

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：非原子性的组合操作。
# 2. 底层原理 (ob_size):
#    - `len(d)` 只是读取结构体的 `ob_size` 字段 (O(1))。
#    - 但读取后，GIL 可能会释放，允许其他线程修改字典。
# 3. 策略：
#    - 在并发环境中，永远不要假设“刚才检查是非空的，现在就一定能取到值”。
#    - 使用 `try-except KeyError` (EAFP 风格) 或者加锁。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：len(d) 是线程安全的吗？
# 策略：“len(d) 本身是线程安全的（读取一个整型）。但基于 len(d) 做的后续决策如果不加锁，就不是线程安全的（TOC/TOU - Time of check to time of use bug）。”
```
#### [Tips 5] 递归字典与 `__repr__` 爆炸

**场景**：打印一个自我引用的字典。
**PHP思维**：var_dump 会检测递归并显示 *RECURSION*。

```python
d = {}
d['self'] = d

# 🚀 推荐 (Safe)
print(d) 
# 输出: {'self': {...}} 
# Python 的 dict.__repr__ 有环检测机制。

# 💀 性能杀手 (Killer) - 自定义 __repr__ 没做好防护
class BadDict(dict):
    def __repr__(self):
        return "BadDict(" + str(self.items()) + ")" # 这里的 items() 转换可能死循环

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：对象转字符串。
# 2. 底层原理 (Py_ReprEnter):
#    - CPython 内部维护了一个 `thread_local` 的递归检查栈。
#    - 标准 `dict` 在 `repr` 时会调用 `Py_ReprEnter` 检查当前对象是否已经在栈中。如果是，直接返回 `"{...}"`。
#    - 如果你重写了 `__repr__` 且没有使用 `reprlib.recursive_repr` 装饰器，就会导致 `RecursionError` 甚至栈溢出。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 替代方案：在自定义复杂对象时，务必使用 `reprlib` 模块来处理递归保护。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：如何安全地打印可能包含环的数据结构？
# 策略：“标准容器自带递归检测。自定义类需要使用 `reprlib.recursive_repr` 装饰器，或者手动维护一个 `seen` 集合。”
```

#### [Tips 6] 字典不仅是哈希表 —— __dict__ 的可写性

**场景**：Monkey Patching (猴子补丁)，动态给对象添加方法。
**PHP思维**：魔术方法 `__call`。

```python
class A:
    def hello(self): print("Hi")

def new_hello(self): print("Hacked")

# 🚀 推荐 (Advanced Metaprogramming)
# 直接修改类的字典
A.hello = new_hello 
# 或者
setattr(A, "hello", new_hello)

# 🏴‍☠️ 黑客/底层 (Direct __dict__ access)
# 有些内置类型（如 int, list, dict）的 __dict__ 是只读代理 (mappingproxy)，不允许直接写入！
# dict.new_method = ... # ❌ TypeError

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：修改类属性。
# 2. 底层原理 (Type Object vs Heap Type):
#    - 用户定义的类（Heap Types）的 `__dict__` 通常是可写的。
#    - C 扩展定义的类（Static Types，如 `list`, `str`）通常是不可变的，或者其 `__dict__` 被 `MappingProxyType` 保护起来，防止 Python 代码破坏 C 层的内存布局。
# 3. 内存动作：
#    - 这解释了为什么你不能给内置 `str` 类增加新方法，但可以给自己写的类增加。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：能给 dict 类添加新方法吗？
# 策略：“不能，因为 dict 是内置类型，其底层结构体大小固定。必须通过继承 UserDict 或 subclassing dict 来扩展。”
```