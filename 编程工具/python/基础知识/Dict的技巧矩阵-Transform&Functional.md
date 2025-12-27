#### [Tips 1] 双列表转字典 (Zipping) —— 拉链操作

**场景**：前端传来了两个数组 `['id', 'name']` 和 `[1, 'Alex']`，你需要把它们拼成对象。
**PHP思维**：array_combine($keys, $values)。

```python
keys = ["id", "name", "role"]
values = [1, "Alex", "Admin"]

# 🐢 较慢 (Slow) - 手动循环
# d = {}
# for i in range(len(keys)):
#     d[keys[i]] = values[i]

# 🚀 推荐 (Fast/Pythonic) - Zip + Dict Constructor
d = dict(zip(keys, values))

# ⚡ 极速 (Instant) - 如果数据量极大，使用 itertools.izip (Python 2) 或 zip (Python 3)
# Python 3 的 zip 返回的是迭代器，不生成中间列表。
# dict() 构造函数直接消费这个迭代器。

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：并行遍历两个迭代器。
# 2. 底层原理 (Zip Object):
#    - `zip` 创建一个轻量级的 zip 对象，只保存两个迭代器的指针。
#    - `dict()` 内部循环调用 `zip_next`，后者分别调用两个列表的 `next`，打包成 Tuple。
#    - 然后 `dict` 直接解析这个 Tuple 插入哈希表。
# 3. 内存动作：
#    - 零中间内存分配（Python 3）。如果是 Python 2 的 `zip()` 会产生一个 List，那是内存杀手。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果两个列表长度不一致，`zip` 会以短的为准截断。如果这不符合预期，数据会丢失。
# 2. 替代方案：`itertools.zip_longest` (填充 None)。
# 3. 权衡：`zip` 是 C 级别的循环，比手动 Python `for` 循环快很多。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：zip 是完全并行的吗？
# 策略：“在逻辑上是并行的。但在 CPython 实现中，它是交替获取 iter1 和 iter2 的元素。它的开销非常低，几乎等同于两次指针跳转。”
```

#### [Tips 2] 字典反转 (Inverting with Multi-values)

**场景**：将 `{'A': 1, 'B': 1, 'C': 2}` 转为 `{1: ['A', 'B'], 2: ['C']}`。
**PHP思维**：手动循环，判断 `isset` 然后 `push`。

```python
from collections import defaultdict

data = {'A': 1, 'B': 1, 'C': 2}

# 🚀 推荐 (Fast/Pythonic)
inverted = defaultdict(list)
for k, v in data.items():
    inverted[v].append(k)

# 🏴‍☠️ 黑客/底层 (Functional Style) - 稍微慢点但极度简洁
# 如果你追求单行代码 (One-liner)
# sorted_items = sorted(data.items(), key=lambda x: x[1])
# from itertools import groupby
# inverted = {k: [x[0] for x in g] for k, g in groupby(sorted_items, key=lambda x: x[1])}

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：分组聚合。
# 2. 底层原理 (Dict Append):
#    - `defaultdict` 避免了 Python 层面的 `if v not in inverted` 检查。
#    - `list.append` 是摊还 O(1) 的，非常快。
# 3. 对比：
#    - `groupby` 版本虽然看着酷，但需要先排序 (O(N log N))，对于未排序数据，性能远不如 `defaultdict` (O(N))。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果 Value 不可哈希（如 List），不能做 Key。需转 Tuple。
# 2. 替代方案：无，`defaultdict` 是标准解法。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：如何实现类似 SQL Group By 的功能？
# 策略：“在 Python 内存中做 Group By，首选 `defaultdict` 或 `pandas`。`itertools.groupby` 有个大坑：它要求数据必须先排序，否则只能聚合相邻的元素。”
```

#### [Tips 3] 字典过滤 (Filtering)

**场景**：移除所有值为 None 的项。
**PHP思维**：array_filter($arr)。

```python
data = {"a": 1, "b": None, "c": 0}

# 🐢 较慢 (Slow) - 复制列表再删除
# for k in list(data):
#     if data[k] is None: del data[k]

# 🚀 推荐 (Fast/Pythonic) - 字典推导式
# 创建新字典，抛弃旧的
clean_data = {k: v for k, v in data.items() if v is not None}

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：条件复制。
# 2. 底层原理 (DICT_ADD):
#    - 推导式编译为紧凑的字节码循环。
#    - 相比于 `del` 操作（涉及哈希表收缩和墓碑标记），新建字典往往更快，特别是当删除比例较大时。
# 3. 内存动作：
#    - 需要短暂的双倍内存（旧字典 + 新字典）。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果字典有 1GB，推导式会瞬间再申请 1GB，可能 OOM。
# 2. 替代方案：如果内存紧张，必须使用 `copy()` 迭代器或生成器，或者退回到“先收集 Key 再 del”的原地修改模式。
# 3. 权衡：时间 vs 空间。推导式是时间最优。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：filter() 函数好用吗？
# 策略：“`dict(filter(lambda...))` 写法在 Python 3 中不如字典推导式快，因为 Lambda 调用有开销，且可读性差。”
```

#### [Tips 4] 字典映射 (Mapping Values)

**场景**：把所有用户的分数乘以 100。
**PHP思维**：`array_map(function($v){ return $v*100; }, $arr)`。

```python
scores = {"u1": 0.5, "u2": 0.8}

# 🚀 推荐 (Fast/Pythonic) - 字典推导式
scaled = {k: v * 100 for k, v in scores.items()}

# 🏴‍☠️ 黑客/底层 (Map Function) - 仅当转换逻辑是 C 函数时
# 比如把所有值转字符串
# str 是内置 C 函数，比 lambda 快
stringified = dict(zip(scores.keys(), map(str, scores.values())))

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：全量转换。
# 2. 底层原理 (Vectorization simulation):
#    - 这里的 `map(str, ...)` 运行在 C 层面，没有 Python 栈帧开销。
#    - 但 `zip` 和 `dict` 的重建成本可能抵消这一优势，除非字典极大。
# 3. 结论：
#    - 通常情况下，推导式是最快且最可读的。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：如果计算非常复杂（如矩阵乘法）。
# 2. 替代方案：**不要用字典**。转用 `numpy` 数组或 `pandas` Series，它们支持 SIMD 指令集并行计算。
# 3. 权衡：字典是通用容器，不是计算容器。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：Python 中 map 和推导式谁快？
# 策略：“如果映射函数是内置 C 函数（如 `str`, `int`），`map` 往往稍微快一点。如果是 Lambda 或自定义 Python 函数，推导式快（因为少了一层 Lambda 包装）。”
```

#### [Tips 5] 极速 JSON 序列化 (High Performance JSON)

**场景**：API 接口返回大字典。
**PHP思维**：`json_encode($arr)` 极快（C 实现）。

```python
import json
data = {i: str(i) for i in range(100000)}

# 🐢 较慢 (Slow) - 标准库
# output = json.dumps(data) 

# 🚀 推荐 (Fast/Pythonic) - orjson / ujson
# 需要 pip install orjson
import orjson
# orjson 返回 bytes，速度通常是标准库的 5-10 倍
output_bytes = orjson.dumps(data)

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：将对象图遍历并序列化为字符串。
# 2. 底层原理 (Rust/C Implementation):
#    - 标准库 `json` 即使是 C 扩展，也处理了很多边缘情况，且产生大量临时小对象。
#    - `orjson` 使用 Rust 编写，内存分配极度优化，且跳过了很多不必要的 Python 对象创建。
#    - 它直接操作底层内存，甚至支持 `numpy` 数组。
# 3. 内存动作：
#    - 标准库可能会在序列化过程中产生大量字符串碎片。`orjson` 几乎是流式的。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：Web 服务 50% 的 CPU 可能都花在 JSON 序列化上。
# 2. 替代方案：无脑替换为 `orjson`。注意它不支持一些非标准类型（如 Decimal，需配置）。
# 3. 权衡：引入第三方二进制依赖。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：标准库 json 为什么慢？
# 策略：“因为它太‘动态’了，且为了兼容性做了很多检查。Rust 实现的 `orjson` 利用了强类型语言的优势和零拷贝技术，是目前的性能天花板。”
```

#### [Tips 6] 自定义序列化 (Custom Encoder)

**场景**：字典里有 datetime 对象，json.dumps 报错。
**PHP思维**：PHP `json_encode` 并不自动处理 DateTime 对象（除非实现了 `JsonSerializable` 接口），通常转字符串。

```python
import json
from datetime import datetime

data = {"time": datetime.now()}

# 🐢 较慢 (Slow) - 预处理
# 先把所有 datetime 遍历转成 str，再 dumps。这需要一次额外的 O(N) 遍历。

# 🚀 推荐 (Fast/Pythonic) - 继承 JSONEncoder
class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

json.dumps(data, cls=DateTimeEncoder)

# ⚡ 极速 (Instant) - orjson 原生支持
import orjson
orjson.dumps(data, option=orjson.OPT_NAIVE_UTC)

# 💡 源码级剖析 (Source Code Analysis)
# ------------------------------------------------------------------
# 1. 动作描述：遇到未知类型时回调。
# 2. 底层原理 (Callback Overhead):
#    - 使用 `cls=...` 会导致 JSON 编码器在 C 代码和 Python 代码之间频繁切换（每次遇到未知对象都要回调 Python 方法），性能下降明显。
#    - `orjson` 在 Rust 层面内置了对 `datetime` 的支持，避免了这种 Context Switch。

# 🔧 优化路径 (Optimization Path)
# ------------------------------------------------------------------
# 1. 瓶颈识别：大量自定义对象序列化。
# 2. 替代方案：在对象内部实现 `__str__` 或转 dict 方法，尽量让序列化库处理基本类型。

# 💡 面试官视角 (Interview Corner)
# ------------------------------------------------------------------
# 问题：如何处理复杂对象的序列化？
# 策略：“优先选支持 Native 处理的库（如 orjson）。如果不行，写 `default` 回调，但要做好性能折损的心理准备。”
```