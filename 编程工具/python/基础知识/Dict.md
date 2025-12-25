## 1. 基础常用
1. **安全获取：** d.get("key", default) 防止 KeyError。
2. **获取键值对：** d.items() 遍历 k, v。
3. **获取所有键/值：** d.keys(), d.values()。
4. **删除并获取：** d.pop("key")。
5. **更新/合并：** d.update(other_dict)。
6. **清空：** d.clear()。
7. **检查 Key：** if "k" in d:。
8. **设置默认值：** d.setdefault("k", []).append(1)。
9. **字面量创建：** {} 比 dict() 快。
10. **长度检查：** len(d)。
## 2. 进阶高级
1. **字典推导式：** {k: v**2 for k, v in data}。
2. **defaultdict：** from collections import defaultdict 自动处理缺失键。
3. **按 Value 排序：** sorted(d.items(), key=lambda x: x[1])。
4. **合并运算符 (Py3.9+)：** new_d = d1 | d2。
5. **Counter：** 词频统计神器，支持加减运算。
6. **popitem()：** Python 3.7+ 保证移除最后插入的项（LIFO）。
7. **倒排索引：** {v: k for k, v in d.items()} (前提是 value 唯一)。
8. **fromkeys：** dict.fromkeys(keys, 0) 快速初始化。
9. **OrderedDict：** 虽然普通 dict 有序，但 OrderedDict 支持 move_to_end 实现 LRU。
10. **字典视图：** keys() 返回的是视图对象，支持集合运算 d1.keys() & d2.keys()。

## 3. 黑客技巧
1. **布尔值 Key 碰撞：** {True: 'yes', 1: 'no'} 结果是 {True: 'no'}。因为 True == 1 且 hash(True) == hash(1)。
2. **__missing__：** 子类化 dict 并定义此方法，自定义 Key 缺失时的行为。
3. **ChainMap：** 逻辑合并多个字典而不复制数据（作用域查找模拟）。
4. **只读字典：** types.MappingProxyType(d) 创建字典的只读视图。
5. **fromkeys 的坑：** dict.fromkeys([1, 2], []) 所有 key 共享同一个列表对象。
6. **Switch Case 模拟：** 用 func_dict.get(case, default_func)() 代替长 if-else。
7. **__dict__ 访问：** obj.__dict__ 获取对象的属性字典（动态修改对象属性）。
8. **内存压缩：** Python 3.6+ 使用了紧凑字典实现（Compact Dict），内存减少 20%-30%。
9. **Hash 攻击防御：** Python 字典使用了随机 Hash 种子防止 DoS 攻击。
10. **WeakValueDictionary：** weakref 模块，允许值被垃圾回收（用于缓存）。