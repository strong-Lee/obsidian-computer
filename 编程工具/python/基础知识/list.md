## 1. 基础常用
1. **负索引访问：** lst[-1] 获取最后一个元素。
2. **切片复制：** new_lst = old_lst[:] 快速浅拷贝。
3. **步长切片：** lst[::2] 每隔一个取一个。
4. **列表反转：** lst[::-1] 快速生成反转列表。
5. **检查存在：** if x in lst: 判断元素是否存在。
6. **追加元素：** lst.append(x) 在末尾添加。
7. **扩展列表：** lst.extend(other_lst) 合并两个列表。
8. **统计次数：** lst.count(x) 统计 x 出现的次数。
9. **获取索引：** lst.index(x) 获取 x 第一次出现的下标。
10. **排序：** lst.sort() (原地排序) 或 sorted(lst) (返回新列表)。
## 2. 进阶高级
1. **列表推导式：** [x**2 for x in range(10) if x % 2 == 0]。 
2. **带索引遍历：** for i, v in enumerate(lst):。
3. **同时遍历多个列表：** for x, y in zip(list1, list2):。
4. **解包赋值：** first, *middle, last = lst (获取首尾，中间打包)。
5. **自定义排序键：** lst.sort(key=lambda x: len(str(x))) 按长度排序。
6. **原地切片赋值：** lst[1:3] = [10, 20, 30] 替换并改变长度。
7. **filter与map：** 配合 lambda 进行函数式编程。
8. **any() 和 all()：** if any(x > 10 for x in lst): 检查条件。
9. **二维列表转一维：** sum(nested_list, []) (仅限小列表，慢) 或 [item for sublist in nested in sublist]。
10. **bisect 模块：** 在有序列表中进行二分查找插入 bisect.insort(lst, x)。

## 3. 黑客技巧
1. **原地清空：** lst[:] = [] 或 lst *= 0 (比 clear() 更底层，保持对象引用不变)。
2. **引用陷阱利用：** row = [0] * 3; matrix = [row] * 3 (修改一个元素会变整列，虽然通常是bug，但有时用于同步状态)。
3. **利用 sys.getsizeof：** 列表预分配内存策略是“超额分配”，观察内存增长规律。
4. **快速转置矩阵：** list(zip(*matrix))。
5. **无限迭代器转列表：** 利用 iter 的哨兵模式 list(iter(func, sentinel))。
6. **__getitem__ 魔法：** 让自定义类像列表一样支持切片。
7. **线程安全性：** append 和 pop 操作在 CPython 中是原子的（线程安全），但 i += 1 不是。
8. **列表作为默认参数的坑：** def f(l=[]): (用于实现函数级缓存)。
9. **深拷贝加速：** 在某些特定结构下，序列化再反序列化（pickle/json）可能比 deepcopy 快。
10. **删除遍历中的元素：** 倒序遍历 for i in range(len(lst)-1, -1, -1) 删除元素，避免索引错位。