---
title: Chapter 1 The Python Data Model
weight: 1
type: docs
---

> dunder method: 双下划线方法（`__getitem__`、`__len__` 等，用 dunder-getitem 表示）

## 1.1 一摞 Python 风格的纸牌

### 具名元组

>  **Factory Function** for Tuples with Named Fields
> `collections.namedtuple`(_typename_, _field_names_, _*_, _rename=False_, _defaults=None_, _module=None_)[](https://docs.python.org/3/library/collections.html#collections.namedtuple "Permalink to this definition")
> Returns a new tuple subclass named _typename_. The new subclass is used to create tuple-like objects that have fields accessible by attribute lookup as well as being indexable and iterable. Instances of the subclass also have a helpful docstring (with typename and field_names) and a helpful `__repr__()` method which lists the tuple contents in a `name=value` format.

- 功能：**只有少数属性，但没有方法的对象**，如数据库条目；
- 用法如下；
- 会在 **2.3** 中具体研究。
```python
from collections import namedtuple
Card = namedtuple('Card', ['rank', 'suit'])
c1 = Card('10', 'spades')
print(c1.rank, c1.suit)
```

### \[\]操作符——取值与切片

- 取值与切片的操作实际上是在同一接口中实现的——`__getitem__(self, idx)`；
- `idx` 中为单一参数时，表示读取值；
- `idx` 中为切片操作时，实际上传入的是一个 slice（切片）对象。会在 **2.4** 中具体研究；
- 如果没有实现 `__iter__` 方法，则在实现了 `__getitem__` 操作之后，**自动成为了可迭代对象**，`for` 迭代的操作使用索引逐个访问实现（可认为 `__iter__` 操作使用这一方式实现）。
- 如果没有实现 `__contains__` 方法，则 `in` 操作会进行迭代搜索。

### 字典

- 这里使用的创建字典的方式：`dict(a=v1, b=v2)`（**一般函数的参数列表就是一个字典**）

### 成员保护和访问限制

- 保护成员数据类型：单下划线 `_item`；
- 私有成员数据类型：双下划线 `__item`；
- 注意：访问的限制与 C++不同：
    - `_name`、`_name_`、`_name__`:**建议性**的私有成员，不要在外部访问。注意：这类成员不可以通过 `from XXX import xxx` 的方式导入；
    - `__name`、 `__name_` :**强制的**私有成员，但是你依然可以蛮横地在外部危险访问。
    - `__name__`:**特殊成员，与私有性质无关**，例如 `__doc__`。

## 1.2 如何使用特殊方法

### 特殊方法的使用

- 特殊方法仅供**解释器**使用，不能自己手动调用。需要使用特定的接口来使用这些方法（除了父类构造器的 `__init__` 需要在子类中调用）；
- 对于内置类型，CPython 会直接处理底层的 C 语言结构体属性，比方法调用更快；
- python 内置 `complex` 类可以表示二维向量；
- `math.hypot()` 返回欧几里得范数$\sqrt{x^2+y^2}$。

### 字符串的形式

- `__repr__`
    - 用于交互式控制台和调试程序的字符串形式；
    - `%r` 调用 `repr` 函数，能返回**标准字符串**的表示形式。**若类型为字符串，则会输出引号**，否则不会。鉴于其使用的情形为交互式环境，因此可以用来区分数据类型。
- `__str__`
    - 在 `str()` 函数和 `print()` 打印时使用；
    - 如果没有实现，会使用 `__repr__` 做替代。因此**至少需要实现 `__repr__`**。

## 1.3 特殊方法一览

参见 Python 语言参考手册中的"Data Model"

《流畅的 Python》P11
