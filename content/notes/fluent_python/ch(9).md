---
title: Chapter 9 A Pythonic Object
weight: 9
type: docs
---

## 9.1 对象表示形式

- `__repr__` 和 `__str__` 特殊方法对应了基本的字符串表示；
- `__bytes__` 返回字节序列表示形式；
- `__format__` 使用特殊的格式代码显示字符串表示形式。

## 9.2 再谈向量类

- `__iter__` 方法的实现将实例变成可迭代对象，这样才能**拆包**。`*self` 就以此为基础。

## 9.3 基于字节序列的备选构造方法

```python
    @classmethod  # <1>  
    def frombytes(cls, octets):  # <2>  
        typecode = chr(octets[0])  # <3>  
        memv = memoryview(octets[1:]).cast(typecode)  # <4>  
        return cls(*memv)  # <5>
```

- 使用**类方法**来定义构造方法：用 `@classmethod` 装饰器装饰。类方法传入的参数为**类本身**`cls`；
- 在其内部使用默认的构造函数时，**调用 `cls()` 构造函数**（`cls` 为传入的参数）即可；
- 技巧：`memoryview` 对象经过 `cast` 转换后，可直接迭代、拆包，无需再使用 `.tolist()` 方法。

## 9.4 `classmethod` 与 `staticmethod`

- `classmethod`：定义**操作类**的方法，**第一个参数为类本身**，常用于备选构造方法。使用 `A.method(*args)` 调用，传入的参数包括类本身 `A` 和 `args`；
- `staticmethod`：就是普通的函数，只是在类中定义。**没有特别的参数要求**。使用 `A.method(*args)` 调用，传入的参数没有 `A`，只有 `args`。

## 9.5 格式化显示

- `format(value, spec)` 和 `str.format(values)` 都是格式化方法，会调用 `.__format__(spec)` 方法。其中 `spec` 是 `str.format(values)` 中格式字符串的 `{}` 里**冒号后的部分**（**单个处理**。如出现多个 `{}`，则 `format` 方法会多次调用这一特殊方法，无需手动实现）；
- **格式字符串句法**（`str.format()`，Format String Syntax）简介：
    - `b` 和 `x` 表示二进制和十六进制；
    - `f` 表示浮点数类型；
    - `%` 表示百分数形式；
    - 以上的规定仅限于内置类型中的实现，**可自定义如何格式化**（可扩展）；

```python
"First, thou shalt count to {0}"  # References first positional argument
"Bring me a {}"                   # Implicitly references the first positional argument
"From {} to {}"                   # Same as "From {0} to {1}"
"My quest is {name}"              # References keyword argument 'name'
"Weight in tons {0.weight}"       # 'weight' attribute of first positional arg
"Units destroyed: {players[0]}"   # First element of keyword argument 'players'.
"Harold's a clever {0!s}"        # Calls str() on the argument first
"Bring out the holy {name!r}"    # Calls repr() on the argument first
"More {!a}"                      # Calls ascii() on the argument first
```

- 如果没有定义 `__format__` 方法，默认会使用 `str()` 作为格式化结果，且**不接受格式说明符**；
- 可根据需求扩展支持的格式代码，自定义格式说明符。

```python
    def __format__(self, fmt_spec=''):  
        if fmt_spec.endswith('p'):  
            fmt_spec = fmt_spec[:-1]  
            coords = (abs(self), self.angle())  
            outer_fmt = '<{}, {}>'  
        else:  
            coords = self  
            outer_fmt = '({}, {})'  
        components = (format(c, fmt_spec) for c in coords)  
        return outer_fmt.format(*components)
```

## 9.6 可散列的 `Vector2d`

- 可用 `@property` 装饰器把读值方法标记为**特性**，这样可以直接读值（不按方法调用）；
- 使用**两个前导下划线**把属性标记为私有的；
- 最好使用**位运算符异或**`^` 混合各分量的散列值；
- 实例的散列值**不应该变化**，因此需要将属性和特性标记为只读的。

## 9.7 Python 的私有属性和“受保护的”属性

- Python 中避免子类意外地覆盖父类的私有属性：会把以 `__var` 命名的属性进行**名称改写**，以 `_ClassName__var` 的形式存入 `__dict__` 中，这样就不会产生属性覆盖的问题；
- 但是这也说明，只要知道这一改写机制，可以**直接读取、修改私有属性**；
- 可以使用**单个下划线前缀**来表示受保护的属性。Python 解释器不会做特殊处理，但程序员约定俗成不会在类外部访问这种属性；
- 不要依赖默认的名称改写方式，而应当自行明确一种方式，这样更易于理解。

## 9.8 使用 `__slots__` 类属性节省空间

- 在类中创建一个**类属性**`__slots__`，将其赋值为字符串构成的可迭代对象，表示**所有实例属性**。这样 Python 会在各实例中使用**类似元组的结构**存储实例变量，避免 `__dict__` 消耗大量内存。这样一来也**无法动态创建属性**；
- `__slots__` 属性不会被继承，因此需要在子类中都定义；
- 把 `__dict__` 加入 `__slots__` 中，会在保存实例属性的同时**支持动态创建属性**，保存在 `__dict__` 中。但这违背了初衷，无法节省内存；
- 如果要支持弱引用，则需要包含 `__weakref__` 属性（默认有），并**将其加入 `__slots__` 中**。

## 9.9 覆盖类属性

- 可以使用实例访问**类属性**：使用与实例属性相同的 `.` 操作读取。但如果试图以此方式赋值，则会**新建实例属性，而把同名类属性覆盖**。该特性可概括为：**类属性可用于为实例属性提供默认值**；
- 要修改类属性，必须**直接使用类对象**，用`.`操作符获取类属性并修改。更符合Python风格的修改方式为**创建子类后在类内修改**；
- 为了提高可扩展性，**不要在类内硬编码类名**，采用`type(self).__name__`的方式可以获取类名。
