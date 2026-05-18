---
title: Chapter 2 Lists and Dictionaries
weight: 2
type: docs
---

## 11. 了解切割序列的方法

- 列表切片**不会出现索引越界的问题**，因此可用来限定最大长度；
- 列表切片复制产生**全新的列表**，仅复制了列表元素的值（引用）；
- 可对切片赋值，此时**无需新值的元素个数相等**；
- 完整列表切片的赋值不会分配新的内存位置，与一般切片赋值一致。

## 12. 在单次切片操作内，不要同时指定 start, end, stride

- 如果一定要配合 start 或 end 来使用 stride，那么可以考虑**步进式切片**，使用多次切片来完成目标；
- `itertools.islice()` 可得到**懒惰切片**，在使用时实时产出切片内容。它不允许使用负值来切片。

## 13. 使用包含 `*` 的拆包来替代切片

- 使用 `a, *others, b = ...` 来拆包数组，避免显式指定索引值导致错误；
- 适用于处理含义不同的元素，如表格的表头行和内容行；
- 适用于处理包含**至少若干个元素**的序列；
- 需要注意**拆包结果的内存超限问题**。

## 14. 使用 `key` 参数指定排序的标准

- 如果要使用多个排序标准（先按 A 属性，再按 B 属性），则可以借助元组的比较规则——按序依次比较直至分出结果，**将返回元组的 lambda 函数作为 `key`**；
- 如果不同排序标准要求的顺逆序不同，则可以借助 Python 排序的**稳定性**，多次调用 `sort` 进行排序，结合不同的排序标准。

## 15. 谨慎依赖字典的插入顺序

- Python 3.5 及以前的版本会乱序迭代字典，与插入顺序不一致。**如果需要依赖字典顺序，建议使用 `OrderedDict`**；
- 如果某一过程需要借助字典的有序性，而实际的字典实例并非原生 `dict` 因而不具有有序性（鸭子类型），则可能出现问题。解决方法包括去除有序性依赖、添加类型判断、添加类型标注（`typing.Dict[..., ...]`）；
- `popitem()` 方法会弹出字典中的一组键值对（无顺序要求）。

## 16. 使用 `get` 来处理字典键缺失，而非 `in` 或 `KeyError`

- 实现计数器的自增操作：
    - 这里看似可以使用 `setdefault` ，但实际上是多余的（一定会在后一行修改值）。

```python
count = counters.get(key, 0)
counters[key] = count + 1
```

- 扩展计数器，每个键对应一个列表：
    - `setdefault` 的方法看似更简洁，但方法名称并不易理解，且每次执行该操作都**必须先创建空列表**，使用其他方法可以避免。

```python
names = votes.get(key)
if names is None:
    votes[key] = names = []
names.append(who)
#### OR ####
if (names := votes.get(key)) is None:
    votes[key] = names = []
names.append(who)
#### OR ####
names = votes.setdefault(key, [])   # not self-explanatory
names.append(who)
```

- 如果真的需要使用 `setdefault`，那么更应该考虑 `defaultdict`。

## 17. 使用 `defaultdict` 而非 `setdefault` 处理缺省键值

## 18. 懂得使用 `__missing__` 来构造依赖于键的缺省值

- 上述方法的局限性：`defaultdict` 的构造函数不支持输入参数，无法根据键的值来执行不同的构造方式；
- 解决方法：**创建 `dict` 的子类，实现 `__missing__(self, key)` 特殊方法，在其中设置缺省值并返回**。该特殊方法会在未查找到键时调用。

