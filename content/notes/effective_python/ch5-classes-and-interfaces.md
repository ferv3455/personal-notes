---
title: Chapter 5 Classes and Interfaces
weight: 5
type: docs
---

## 37. 尽量用辅助类来维护程序的状态，而不要用字典或元组

- `dict.setdefault(key, default=None)` 可以更安全地初始化字典键值，**仅在不存在时添加**；
- 当状态内部嵌套层数多余一层时（包含字典的字典），就需要避免这种情况，**将嵌套结构重构为类**，提供明确的接口；
- `collections.namedtuple`**（具名元组）适合定义精简而不可变的数据类**，如单次考试的成绩信息。
  - 局限性：无法指定参数默认值，且实例的属性仍然可以通过下标访问；
  - 可以在需要时（定义行为等）修改为完整的类，而使用相同的定义接口。

## 38. 简单的接口应该接受函数，而不是类的实例

- `defaultdict` 在定义时需要传入函数来定义缺省值初始化方式，这是**挂钩（hook）函数**，提供这样的函数有利于分隔函数中的附带效果与确定的行为（如定义闭包作为参数等）；
- 如果要在函数内部保存状态，则可以**定义一个实现了可调用接口（`__call__`）的类，将其作为挂钩传入**，而不使用闭包来实现。

## 39. 以 `@classmethod` 形式的多态去通用地构建对象

- 通过 `@classmethod` 机制，可以使用参数中的 `cls` 来构造类的对象，以此实现**构造函数的重载**；
- 类方法可以实现多态，这样可以以更通用的方式灵活构建子类。

```python
class GenericInputData(object):
    def read(self):
        raise NotImplementedError

    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

class PathInputData(GenericInputData):
    def __int__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        return open(self.path).read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))

class GenericWorker(object):
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers

class LineCountWorker(GenericWorker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result

def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)

mapreduce(LineCountWorker, PathInputData, config)
```

## 40. 用 `super` 初始化父类

- **MRO** 用标准的流程来安排超类之间的初始化顺序（深度优先、从左至右等），可调用类对象的 `mro()` 方法查看（实际初始化的顺序从下至上，再从上至下）；
- 总是应该使用 `super()` 初始化父类；
- `super()` 的默认参数为 `__class__, self`。

## 41. 只在使用 Mix-in 组件制作工具类时进行多重继承

- **Mix-in 类**只定义了其他类可能需要提供的一套附加方法，而不定义自己的实例属性，也不用来构造实例，仅仅用于通过继承来给其他类添加方法。最好将各功能实现为可插拔的 mix-in 组件，相关的类只需要继承需要的那些组件即可；
- Mix-in 类中可以直接使用未定义的变量，通过**动态检测机制**来在运行时绑定实例的变量；
- Mix-in 类中定义的方法也可以在子类中覆盖，修改这些附加方法的表现。

```python
class JsonMixin(object):
    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data)
        return cls(**kwargs)

    def to_json(self):
        return json.dumps(self.to_dict())
```

## 42. 多用 public 属性，少用 private 属性

- Python 中只包含 public, private 两种可见度，private 仅当前类内部可以访问（子类无法访问，其原因在于 private 变量的真实名称不同）；
- 类级别的方法也声明在 class 内，因此**也可访问 private 属性**；
- private 级别变量的实际名称为 `_CLASS__FIELD`；
- **应该多用 protected 属性，在文档中标注字段的合理用法，而不要使用 private 来限制访问**（否则如果必须要在子类中访问，则会遇到麻烦）；
- 在**设计超类**时（尤其是公共 API），为了避免子类的属性名与超类重复（子类中的属性名不受自己控制），可以在超类中使用 private 属性。

## 43. 继承 `collections.abc` 以实现自定义的容器类型

- 在实现了  `__getitem__`  操作之后，**自动成为了可迭代对象**（会永久迭代直到 `StopIteration`）；
- `in` 操作符会在没有实现 `__contains__` 方法时，**默认使用迭代的方式搜索**；
- `collections.abc` 模块定义了一系列抽象基类，提供了不同容器类型的常用方法，同时给出了必须实现的基础方法。如果子类没有实现基础方法，则会**在创建类的实例时直接抛出异常**（在 `__init__` 时进行检测）。**剩余的方法正是基于这些基础方法来实现**。

