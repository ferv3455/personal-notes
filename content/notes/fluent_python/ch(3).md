---
title: Chapter 3 Dictionaries and Sets
weight: 3
type: docs
---

- 各种实例的所有参数均会存储在`__dict__`字典中；`__builtins__.__dict__`存储所有内置函数。

## 3.1 泛映射类型

- 字典的抽象基类：`collections.abc.Mapping`、`collections.abc.MutableMapping`，可以使用`isinstance`函数来观察到字典或其它映射类型是这些的子类；
- 只有**可散列**的数据类型才能用作映射中的**键**：
	- 需要实现`__hash__()`、`__eq__()`方法，且整个生命周期内该值不变；
	- 原子不可变类型可散列，不可变集合`frozenset`可散列，元组内所有元素不可变时可散列；
	- 判断方式：与散列值相关的对象**内部状态是否都不可变**；
	- 一般定义的对象散列值即为`id()`返回值；
- 构造字典的方式：

```python
a = dict(one=1, two=2, three=3)                    # parameters
b = {'one': 1, 'two': 2, 'three': 3}               # simple
c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))  # zip
d = dict([('two', 2), ('one', 1), ('three', 3)])   # tuples
e = dict({'three': 3, 'one': 1, 'two': 2})         # another dict
f = dict({'one': 1, 'three': 3}, two=2)            # separate
a == b == c == d == e == f
```

## 3.2 字典推导

- 与列表推导类似：`a = {x: y for x, y in codes}`。

## 3.3 映射方法

- `update(m, [***kwargs])`可以接受以下的参数，更新字典，覆盖相同的键：
	- 另一个字典；
	- 包含键值对的可迭代对象；
	- 直接在参数中使用参数列表的形式赋值。
- `setdefault(key, [default])`方法可以**获取不一定存在的键值**。如果查找不到，会将`key`和`default`放入映射中，并返回这个`default`；否则直接返回查找到的内容。

## 3.4 映射的弹性键查询

> 可以通过两种方法解决缺失键查询的问题：`defaultdict`、`__missing__`方法。

### `defaultdict`

- 需要在实例化`defaultdict`时，传入一个**可调用对象**，存入`default_factory`成员变量，在找不到键时会调用它生成值；
- 这样就可以避免使用`setdefault`方法，直接使用`d[key]`来获取键值——即使它原来不存在；
- `default_factory`**只会在`__getitem__`中被调用**，不会在`get`等操作中用到；
- `defaultdict`和`deque`两个数据结构是**底层实现的**，因此逻辑上与纯Python实现的`Mapping`不同。**虽然`defaultdict`是子类，但覆盖了所有方法**。`Mapping.get`的实现是基于`__getitem__`的（因此有后面的`__missing__`方法），因此可以**无需单独实现**，而`defaultdict.get`、`dict.get`不行。

### `__missing__`特殊方法

- 在`__getitem__`找不到键时，Python会调用`__missing__`方法（如果实现），而非抛出异常。它也只会在`__getitem__`中调用；
- 自定义映射类型更适合继承`collections.UserDict`类而非`dict`，具体见**3.6**；
- `dict.keys()`的返回值是一个视图（Python 3）（Dictionary view objects），在其中查找元素的速度很快。

```python
import collections  

class StrKeyDict0(dict):  # <1>  

    def __missing__(self, key):  
        if isinstance(key, str):  # <2>                # 需要考虑递归调用
            raise KeyError(key)  
        return self[str(key)]  # <3>  
  
    def get(self, key, default=None):  
        try:  
            return self[key]  # <4>  
        except KeyError:  
            return default  # <5>  
  
    def __contains__(self, key):  
        return key in self.keys() or str(key) in self.keys()  # <6> # 这里不能直接用 in self
```

## 3.5 字典的变种

- `collections.OrderedDict`在添加键时保持顺序，有相同的迭代次序，可以此弹出首末元素；
- `collections.ChainMap`可容纳多个映射对象，可在查找键值时逐个查找；
- `collections.Counter`给键准备整数计数器，可用作计数器、多重集合；
- `collections.UserDict`用**纯Python实现**了标准`dict`，用于**自定义继承类**。

## 3.6 子类化 `UserDict`

- 继承自`MutableMapping`
- 使用`UserDict`自定义继承类的优点：**避免`dict`中实现时采用的捷径**，具体见**12.1**；
- 具体的存储机制：使用`data`的属性（**是`dict`实例**）来存储数据，以此来避免无限递归调用；
- 其他继承得到、自动实现的方法：
	- `MutableMapping.update`方法基于`__setitem__`方法实现；
	- `Mapping.get`方法基于`__getitem__`方法实现，因此会使用`__missing__`。**在`dict.get`、`defaultdict.get`的实现中，使用了底层途径来绕过`__getitem__`，因此在上面的实现中需要重写`get`方法；**
	- 从以上可以看出，`UserDict`的**可扩展性更强**。

```python
import collections  
  
class StrKeyDict(collections.UserDict):  # <1>  
  
    def __missing__(self, key):  # <2>  
        if isinstance(key, str):  
            raise KeyError(key)  
        return self[str(key)]  
  
    def __contains__(self, key):  
        return str(key) in self.data  # <3>   这里调用的是dict的__contains__方法，不会递归调用
  
    def __setitem__(self, key, item):  
        self.data[str(key)] = item   # <4>    这里调用的是dict的__setitem__方法，不会递归调用
```

## 3.7 不可变映射类型

- 封装类`types.MappingProxyType`作用在映射上只会得到一个映射视图，是**动态的只读视图**；
- 使用`MappingProxyType(d)`得到字典的动态只读映射，可以利用原字典实例`d`继续修改，但无法通过该只读实例修改。

## 3.8 集合

### 概述

- 集合的抽象基类：`collections.abc.Set`、`collections.abc.MutableSet`；
- 集合可用于**去重**；
- 集合中的元素必须是**可散列的**（这是因为其实现和字典一样）。**`set`是不可散列的，`frozenset`是可散列的**，因此可以创建包含`frozenset`的`set`。

### 集合字面量

- 使用`{1}, {1,2}`等**字面量**形式创建集合，空集必须要使用`set()`（否则是空字典）；
- `frozenset`没有特殊字面量句法，只能使用构造（函数）方式；
- 支持使用**集合推导**的形式创建集合。

```python
from unicodedata import name
print({chr(i) for i in range(32, 256) if 'SIGN' in name(chr(i), '')}) # Unicode名字中包含SIGN
```

### 集合操作

- 数学运算。以下的操作中，中缀操作符要求两侧都为**集合**，其他方法可对任意**可迭代对象**操作：
	- 交集`&`（`__and__`）、`&=`（`__iand__`）、`s.intersection`、`s.intersection_update`；
	- 并集`|`（`__or__`）、`|=`（`__ior__`）、`s.union`、`s.update`；
	- 差集`-`（`__sub__`）、`-=`（`__isub__`）、`s.difference`、`s.difference_update`；
	- 对称差（异或）`^`（`__xor__`）、`^=`（`__ixor__`）、`s.symmetric_difference`、`s.symmetric_difference_update`；
	- 属于`in`；
	- 子集`<, <=`、`s.issubset`；
	- 超集`>, >=`、`s.issuperset`；
- 集合也支持其他序列类型操作的方法，只是方法名称有所不同：`add`、`discard`、`pop`（随机）。

## 3.9 背后的实现

### 散列表

- 在字典的散列表中，每个键值对占据一个表元，包含对键和值的引用，通过偏移量读取。集合的表元中仅包含键的引用；
- 如果两个对象在比较时是相等的，那么散列值必须相等。而**越是相似但不相等的对象，散列值差别越大**（在索引空间尽量分散）；
- **“加盐”**：在`str, bytes, datetime`等对象的散列值中加入**与进程相关**的常量，防止DOS攻击；
- **散列冲突**的处理：在散列值中另外取几位，用特殊的打乱方法处理，在把新的数字作为索引。

### 字典、集合的实现

- **键必须可散列**，且散列值不变，相等的键散列值必定要相等（在自定义`__eq__`方法的情况下，需要注意这一点。**如果可变的类实现了`__eq__`，则其实例不应当可散列**）；
- 字典用大量空间换取时间。在**9.8**中提到的`__slots__`属性可改变实例属性的存储方式为元组；
- 键的次序取决于添加顺序（散列冲突）；
- 输出字典时，其顺序是各键**在散列表中的存储顺序**，并非无序；
- 添加新键时，可能会**改变原有键的顺序**（扩容后重新选取散列值）。因此**不能在迭代中修改**；
- `.keys()`、`.items()`、`.values()`返回的是**字典视图**，而非列表，因此是**动态的**。

