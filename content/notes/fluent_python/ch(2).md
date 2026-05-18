---
title: Chapter 2 An Array of Sequences
weight: 2
type: docs
---

## 2.1 内置序列类型概览

- 根据支持的数据类型分类：
    - **容器序列**：能存放不同数据，存放的是不同**对象的引用**，如列表、元组、双端队列 `deque` 等；
    - **扁平序列**：仅支持一种类型，存放的是**值**，如字符串、`bytes`、`bytearray`、`memoryview`、`array.array` 等（后几种序列类型会在后续介绍），是**连续的内存空间**。
- 根据是否能被修改来分类：
    - **可变序列**：列表、`bytearray`、`array.array`、`deque`、`memoryview`；
    - **不可变序列**：元组、字符串、`bytes`；
    - 可变序列是不可变序列的子类，增加了修改相关的方法。

## 2.2 列表推导和生成器表达式

> 列表推导：list comprehension，简称 listcomps。
> 生成器表达式：generator expression，简称 genexps。

### 列表推导

- 使用原则：只用列表推导来**创建新的列表**，并尽量保持**简短**；
- Python 会**忽略代码中 `[] {} ()` 中的换行**，在它们中间可以省略换行符 `\`；
- 在 Python2 中列表推导中的变量会影响上下文，在 Python3 中有了局部作用域；
- `map(func, iterable)` 和 `filter(bool_func, iterable)` 联合使用的功能可以使用列表推导替代，这两个函数会在**第五章**中讨论；
- 可以在列表推导中使用 `for` 迭代，计算笛卡儿积：**嵌套关系与 `for` 语句顺序一致**。

### 生成器表达式

- **生成器表达式用来生成列表以外的序列类型；**
- 使用迭代器协议，逐个产出元素，而非先存储到完整的列表中，更节省内存；
- 生成器的工作原理见**第十四章**。

## 2.3 元组

### 元组的使用

- 基本用途：不可变的列表、**没有字段名的记录**；
- 元组**拆包**的应用：`a,b = tuple_x`（**平行赋值**，常被用于处理返回值）、字符串格式化中的 `%` 符号等，可以运用到**任何可迭代对象**上，唯一要求是数量一致；
- `*` 符号的作用：
    - 把可迭代对象拆开成函数的参数：`divmod(*t)`；
    - 处理**剩下的元素**（如 `*args`）：`a, b, *rest = range(5)`，可出现在任何位置，`rest` 存储列表；
- 可以使用**嵌套结构**进行拆包，只需要符合表达式本身结构即可。

```python
metro_areas = [  
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),   # <1>  
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),  
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),  
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),  
    ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),  
]  
  
print('{:15} | {:^9} | {:^9}'.format('', 'lat.', 'long.'))  
fmt = '{:15} | {:9.4f} | {:9.4f}'  
for name, cc, pop, (latitude, longitude) in metro_areas:  # <2>  
    if longitude <= 0:  # <3>  
        print(fmt.format(name, latitude, longitude))
```

### 具名元组

> 具名元组使得我们可以给记录中的字段命名——这基于元组的用途之一。

- 具名元组创建的类的实例所消耗的**内存与元组相同**，字段名存在对应的类里面；其大小**比普通的对象实例更小**，因为不存在**实例的字典 `__dict__` 来存放各属性**；
- 创建方法：
    - 类名、各字段名构成的**可迭代对象**；
    - 类名、**由空格隔开的各字段名（一个字符串）**；
- 可以通过**字段名**、**索引位置**两种方法来获取字段信息；
- 与元组不同的专有属性：
    - `_fields` **类属性**：返回包含所有字段名称的元组；
    - `_make(iterable)` **类方法**：使用可迭代对象生成实例，和传入拆包等效；
    - `_asdict()` **实例方法**：返回一个**有序字典** `OrderedDict` 实例。

## 2.4 切片

- `a:b:c` 在索引中返回**切片对象**：`slice(a,b,c)`（可以使用的默认参数一致），将其传入 `__getitem__` 的参数；
- 预先创建合适的切片对象有助于提高代码可读性；
- 可以直接**使用可迭代对象对切片赋值**、删除切片等；
- `numpy`等库中的数据类型适用多维切片，`__getitem__`接受元组；
- **省略**`...`可用于函数的参数，也可用于外部库（`numpy`）中的多维切片：`x[i,...] = x[i,:,:,:]`。这一设计用于自定义类、扩展功能；
- 具体实现方法见**10.4**。

## 2.5 对序列使用+和\*

- `+`和`*`均不改变原有操作对象，返回新对象；
- 如果序列中元素为可变对象的引用，则使用`*`复制时，复制的是对象的**同一引用**；
- 建立二维列表时，需要使用列表推导，而非仅用`*`进行复制（**注意：对可变对象的操作均为对其引用的操作，除非显示使用`copy`等进行复制**）；
- 可变对象相关的内容见**第八章**。

## 2.6 序列的增量赋值

- `+=`、`*=`符号均为原地操作，对应`__iadd__`、`__imul__`特殊方法。如果没有实现，则会退一步调用`__add__`和`__mul__`。因此这就涉及到**是否会产生新的对象，还是在原对象上操作**；
- 可变序列一般支持原地操作，不可变序列无法原地操作，因此对不可变序列使用增量赋值必定会变为新的对象；
- **字符串是例外**：CPython 对其进行优化，直接在原有的可扩展空间内操作；
- **不要在（不可变）元组中存储可变对象（列表等）**。由于增量赋值不是原子操作（取对象引用、执行增量复制、将该引用复制回去），会在最后一步遇到问题，也就是说会在赋值成功后报错。

## 2.7 `list.sort`方法和内置函数`sorted`

- 前者就地排序（返回`None`），后者返回有序列表；
- 参数`reverse=False`，表示是否降序；
- 参数`key=I`默认为恒等函数，可以用来指定对比的关键字（以函数表示）。该参数也在`min`、`max`中使用到；
- Python的排序算法**Timsort**是**稳定排序**。

## 2.8 `bisect`处理有序序列

- `bisect.bisect(haystack, needle)`二分查找，返回的位置前面的值都**小于等于**`needle`；
	- 可选参数`lo, hi`：缩小搜寻的范围；
	- `bisect.bisect_left`返回的位置会在**相同元素的前面**，而原函数在相同元素之后；
	- **技巧：建立分数和成绩的区间对应关系时，可以使用查找法**。
- `bisect.insort(seq, item)`直接将元素插入到序列中。

## 2.9 其他数组数据结构

### 数组 array

- 适用范围：**只包含数字的列表**，动态可变长度；
- 创建：`array.array(type, iterable)`
	- 类型：单个字符表示，如`b`表示字符，`d`表示浮点数，`h`表示短整型，小写、大写表示有无符号。可用`array.typecode`获取；
- `array.tofile`和`array.fromfile`接受文件描述符作为参数，其中`fromfile`还需要**指定数据量**。这两种方式存储/读取二进制文件，**速度更快，占用空间也更小**；
- 可以使用`==`操作符来**比较两个数组、列表等是否完全相同**，而`is`操作符用来判断是否是同一实例对象。这与java等语言不同（`===`）。

### 内存视图 memoryview

- 适用范围：**不复制内容，在不同的数据结构间共享内存**，提高空间利用率。也就是说，可以将同一片内存视为不同的数据结构，按不同的方式处理；
- **注意：使用的内存存储方式为小端（Little Endian），高位在后面，低位在前面**；
- 使用该特性修改数组中的某个字节（ 00000000 00000000 改为 00000000 00000100，高位在后面，因此变为 1024）：

```python
import array

numbers = array.array('h', [-2, -1, 0, 1, 2])
memv = memoryview(numbers)
print(memv.tolist())        # [-2, -1, 0, 1, 2]

memv_oct = memv.cast('B')
print(memv_oct.tolist())    # [254, 255, 255, 255, 0, 0, 1, 0, 2, 0]

memv_oct[5] = 4
print(numbers)              # array('h', [-2, -1, 1024, 1, 2])
```

### NumPy 和 SciPy

- SciPy 基于 NumPy，提供了线性代数、数值积分、统计学等方面的专业方法；
- `numpy.loadtxt(filename)`可以直接从文本文件中读取数据；
- `numpy.save(filename, ndarray)`和`numpy.load(filename, mmap_mode)`分别用来存储、读取二进制数据文件（.npy格式）。`mmap_mode`设置内存映射的机制，节省内存。具体见下面的注释；
- *其他技巧：*
	- `time.perf_counter()`可以得到精确计时器的当前时间（秒）。

> **mmap_mode**{None, ‘r+’, ‘r’, ‘w+’, ‘c’}, optional
> If not None, then memory-map the file, using the given mode (see [`numpy.memmap`](https://numpy.org/doc/stable/reference/generated/numpy.memmap.html#numpy.memmap "numpy.memmap") for a detailed description of the modes). A memory-mapped array is kept on disk. However, it can be accessed and sliced like any ndarray. Memory mapping is especially useful for accessing small fragments of large files without reading the entire file into memory.

### 双向队列 collections.deque

- `collections.deque`类双向队列是线程安全的数据类型，不用担心资源锁问题；
- 可以**指定队列的大小，自动在另一端删除元素**；
- 常用操作：
	- `deque(iterable, maxlen=10)`创建双向队列；
	- `dq.rotate(n)`移动最右边的n个元素到左边（n>0），否则移动最左边的到右边；
	- `dq.append(item)`、`dq.appendleft(item)`添加元素到右边、左边；
	- `dq.extend(iterable)`、`dq.extendleft(iterable)`添加到右边、左边（**逐个添加**）；
- 双向队列对**队列头尾的操作进行了优化**，从中间删除会稍慢一些；
- 单个元素添加、弹出均是原子操作，不需要**资源锁**。

### 其他队列

- `queue.Queue`、`queue.LifoQueue`和`queue.PriorityQueue`都是线程安全的类，用于线程间通信。在满员时，会**锁住该线程，直至另外的线程腾出了位置**；
- `multiprocessing`中实现了类似的`Queue`，用于进程间通信；
- `asyncio`用于异步编程；
- `heapq`可以把可变序列当作堆队列或者优先队列。
