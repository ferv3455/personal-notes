---
title: Chapter 5 First-Class Functions
weight: 5
type: docs
---

> 一等函数：在 Python 中，函数是一等对象。

## 5.1 把函数视作对象

- 函数都是**function** 类的实例；
- 函数可以用作赋值，也可以作为参数传给函数，这证明了其“一等”本性；
- `__doc__` 属性是函数对象中的一个属性，使用 `help()` 函数可以返回其内容。

## 5.2 高阶函数：`map`、`filter`、`reduce` 的替代

> 高阶函数：接受函数为参数，或把函数作为结果返回的函数。例如 `map`、`sorted`（**2.7 **节）。

- `map` 和 `filter` 返回生成器，因此可以使用生成器表达式替代（Python2 中为列表）；
- `reduce`（不再是内置函数，在 `functools` 中）是**归约函数**，将某个操作应用到序列的元素上，累计之前的结果，把一系列值归约成一个值。它常用于求和，基本**可用 `sum` 替代**（见**10.6 **节）；
- 归约函数还包括：`all`（全部合取）、`any`（全部析取）等，具体见**14.11 **节。

## 5.3 匿名函数

- `lambda` 函数的定义体只能使用**纯表达式**；
- `lambda` 句法只是**语法糖**，会和 `def` 一样创建函数对象。

## 5.4 可调用对象

- 可以使用 `callable()` 判断对象是否可调用；
- 可调用对象包括：
    - 定义的函数；
    - 内置函数、内置方法（**使用 C 语言实现，经过优化**）；
    - 方法；
    - 类（**类也是对象**，调用时会运行 `__new__` 类方法创建实例，后用 `__init__` 方法初始化）；
    - 定义了 `__call__` 方法的类的实例对象；
    - **生成器函数**（返回**生成器对象**），在**第 14 章**中具体讨论，在**第 16 章**中讨论用作协程。

## 5.5 自定义可调用类型

- 需要在内部维护一个状态，让它在调用之间可用；
- 其他包含内部状态的函数——闭包和装饰器，在**第 7 章**讨论。

## 5.6 函数内省

- 使用 `dir()` 可以得到对象的所有属性名；
- 可以**给函数对象赋予属性**：

```python
def func():
    return 0

func.description = '1'
print(func.description)   # 1
```

- 函数对象**特有**的属性（不仅限于）：
    - `__annotations__`：对**参数和返回值**的注解；
    - `__call__`：实现调用方法；
    - `__code__`：**函数元数据、函数定义体**（字节码）；
    - `__defaults__`：形参的**默认值**；
    - `__globals__`：全局变量；
    - `__name__`：函数名称；
- **IDE 和框架使用 `__defaults__`、`__code__`、`__annotations__` 属性提取函数签名信息。**

## 5.7 定位参数、仅限关键字参数

- 调用函数时使用 `*`、`**` 展开可迭代对象，映射到单个参数。前者接收传入的**定位参数**，后者接收传入的**关键字参数**，都**不可以使用关键字直接赋值**；
- 在 `*args` 表示的参数之后，所有的参数都不可以使用定位参数传入，否则会被 `args` 捕获。这种参数就被称作**仅限关键字参数**；
- 如果不想支持数量不定的定位参数，而想支持仅限关键字参数，**在签名中放一个 `*` 即可**，它后面定义的参数都是仅限关键字参数。

## 5.8 获取关于参数的信息

### `__defaults__`、`__code__`、`__annotations__` 属性

- 函数对象的 `__defaults__` 属性是一个元组，里面保存**定位参数和关键字参数的默认值**；
- `__kwdefaults__` 属性保存**仅限关键字参数的默认值**；
- **参数的名称**储存在 `__code__` 属性中，它是一个 `code` 对象的引用。其中的 `co_varnames` 元组包含了所有变量名（包括函数体内定义的局部变量），`co_argcount` 为参数个数；
- 可以利用 `co_argcount` 和 `co_varnames` 确定参数的名称，随后从后往前对应 `__defaults__` 中的值（**有默认值的定长参数必须在无默认值的定长参数之后**），由此得到参数默认值。这里的参数名称列表**不包含前缀为 `*`、`**` 的变长参数**，因此在复杂情况下难以确定。

```python
print(tag.__defaults__)
print(tag.__kwdefaults__)
print(tag.__code__.co_varnames)
print(tag.__code__.co_argcount)
print(tag.__code__.co_kwonlyargcount)

'''
None
{'cls': None}
('name', 'cls', 'content', 'attrs')
1
1
'''
```

### `inspect` 模块

- 使用 `sig = inspect.signature(func)` 建立**函数签名**；
- `sig.parameters` 包含**各参数的有序字典**，键为参数名，值为 `inspect.Parameter` 对象，可以使用 `param.name`、`param.kind`、`param.default`、`param.annotation` 得到参数名、**种类**（定位/关键字参数、定位参数元组、关键字参数字典、仅关键字参数、*仅定位参数*）、**默认值**（无默认值为 `inspect._empty`）和参数的**注解**（见下一节）；
- `sig.bind()` 方法接受的**参数匹配方式与函数完全一致**（函数调用怎么传参，这里就怎么传），可以用此方法在调用函数前验证参数。返回 `inspect.BoundArguments` 对象，其 `argument` 属性包含**各形参对应实参值的有序字典**。这种方式与 Python 解释器的机制相同；
- `sig.return_annotation` 属性保存**返回值的注解**。

```python
import inspect

sig = inspect.signature(tag)
print(sig.parameters)

my_tag = {'name': 'img', 'title': 'Sunset Boulevard', 'src': 'sunset.jpg', 'cls': 'framed'}
bound_args = sig.bind(**my_tag)
print(bound_args.arguments)

del my_tag['name']
bound_args = sig.bind(**my_tag)

'''
OrderedDict([('name', <Parameter "name">), ('content', <Parameter "*content">), ('cls', <Parameter "cls=None">), ('attrs', <Parameter "**attrs">)])

OrderedDict([('name', 'img'), ('cls', 'framed'), ('attrs', {'title': 'Sunset Boulevard', 'src': 'sunset.jpg'})])

**TypeError**      Traceback (most recent call last)
...
**TypeError**: missing a required argument: 'name'
'''
```

## 5.9 函数注解

- 可以为函数声明中的参数和返回值添加元数据：添加**注解**。注解表达式可以是**任意类型的**，一般使用**类**和**字符串**；
- 为参数添加注解：在参数后**添加 `: ` 符号和表达式**。有默认值时放在**参数名和等号之间**；
- 为返回值添加注解：**在 `)` 和 `: ` 之间添加 `->` 和表达式**；
- 注解会直接存储在函数的字典属性 `__annotations__` 中（保留注解类型），返回值为 `'return'`；
- 注解对 Python 解释器没有任何意义，只为 IDE、框架等工具的**静态类型检查**功能提供信息。

## 5.10 支持函数式编程的包

### `operator` 模块

> `operator` 模块提供了大部分简单运算、读取操作的函数形式。

- **算术运算符**的函数：`operator.add`、`operator.mul` 等；
- **元素或属性读取**的函数：`operator.itemgetter`、`operator.attrgetter`，需要先**构造函数**：
    - `operator.itemgetter` 基于 `[]` 实现，从序列中获取指定位置的元素。构造时支持传入多个参数，它构建的函数会返回对应位置元素的元组；
    - `operator.attrgetter` 能根据名称提取对象的属性。构造时的参数为属性的名称，支持传入多个参数，支持**深入嵌套对象**（包含 `.` 符号）。
- **调用对象方法**的函数：`methodcaller`，需要先**构造函数**。构造时的第一个参数为方法的名称，后几个参数为**对应方法中的参数**。

### `functools.partial` 冻结参数

- `partial`用于**冻结部分参数**，这样在后续调用函数时，使用的参数更少。如将二元函数（乘积）变为一元函数（乘以3）：`triple = partial(mul, 3)`；
	- 第一个参数是可调用对象；
	- 后面跟着任意个要绑定的定位参数和关键字参数；
	- 该函数返回一个`functools.partial`对象，可以使用`func`、`args`、`keywords`访问原函数和参数；
- `partialmethod`可**在类内定义**，用来以类似的方式处理**方法**，可以直接调用冻结参数后的方法。
