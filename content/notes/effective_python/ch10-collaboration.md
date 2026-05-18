---
title: Chapter 10 Collaboration
weight: 10
type: docs
---

## 82. 了解在哪里找到社区开发的模块

- https://pypi.org （Python Package Index）是 Python 的中心模块仓库，由 Python 社区维护；
- pip: pip installs packages；
- PyPI 中的模块需要有软件许可证，大部分使用免费/开源许可证，允许拷贝模块代码。

## 83. 用虚拟环境隔离项目，并重建其依赖关系

- `pip show <name>` 可以查看软件包的版本与依赖关系；
- `pip freeze > requirements.txt` 可用来保存开发环境对软件包的依赖关系；
- 同一时刻只能把模块的某一个版本安装为全局版本，因此依赖性的传递可能导致两个模块无法同时支持；
- `python -c '<sentence>'` 可以验证某一语句能否在环境中正确执行；
- **不要移动环境目录，所有的路径都以硬编码的形式写在安装目录之中**。

## 84. 为每个函数、类和模块编写文档字符串

- 函数、类和模块都应添加相关联的**文档字符串**。函数的 `__doc__` 特殊属性储存了该函数的文档 docstring；
- 添加文档字符串就定义了 `help` 函数的内容，便于程序开发与测试，同时也可用于 Sphinx 、Read the Docs 等**文档生成工具**；
- 文档字符串的编写规范（PEP 257）：
    - 每个模块都应有**顶级的 docstring**，用三重双引号括起来，作为源文件的**第一条语句**。第一行是一句话，描述本模块的用途；后面的一段话包含操作使用部分的细节信息；之后可以强调本模块中的重要类与函数；
    - 每个类应该有类级别的 docstring，写法与模块级 docstring 大致相同，还应强调**如何与 protected 属性及超类方法相交互**；
    - 每个 public 函数及方法都应该有 doctring，应标明具体的行为、参数、返回值、可能抛出的**异常**。

```python
"""Library for test words for various linguistic patterns.

Testing how words relate to each other can be tricky 
sometimes! This module provides easy ways to determine 
when words you've found have special properties.

Available functions:
- palindrome: Determine if a word is a palindrome.
- check_anagram: Determine if two words are anagrams.
...
"""

class Player:
    """Represents a player of the game.

    Subclasses may override the 'tick' method to provide 
    custom animations for the player's movement depending 
    on their power level. etc.

    Public attributes:
    - power: Unused power-ups (float between 0 and 1).
    - coins: Coins found during the level (integer).
    """
    # ...
    pass

def find_anagrms(word, dictionary):
    """Find all anagrams for a word.

    This function only runs as fast as the test for 
    membership in the 'dictionary' container. It will 
    be slow if the dictionary is a list and fast if 
    it's a set.

    Args:
        word: String of the target word.
        dictionary: Container with all strings that 
            are known to be actual words.
    Returns:
        List of anagrams that were found. Empty if 
        none were found.
    """
    # ...
    pass
```

- 函数 docstring 的一些特例：
    - 如果很简单（无参数、返回简单值），则仅需一句话描述；
    - 如果没有返回值、异常，就不需要提到；
    - 如果包含可变位置参数、关键字参数，就需要具体描述用途；
    - 应当指出参数的默认值；
    - 如果函数是生成器，应当描述迭代时产出的值；
    - **如果函数是协程，应当描述产出的值、希望获取的传入值（`yield` 语句）、停止迭代的条件**。
- 可以**为函数添加注解**，并去掉文档字符串中对变量、返回值类型的要求。

```python
def find_anagrams(word: str,
                  dictionary: Container[str]) -> List[str]:
    """Find all anagrams for a word.

    This function only runs as fast as the test for
    membership in the 'dictionary' container.

    Args:
        word: Target word.
        dictionary: All known actual words.

    Returns:
        Anagrams that were found.
    """
    pass
```

## 85. 用包来安排模块，并提供稳固的 API

- 包：含有其他模块的模块，通过**在目录中添加 `__init__.py` 文件**来定义；
- 包的两大用途：划分名称空间、提供稳固的 API（隐藏内部结构）；
- 模块中的 `__all__` 特殊属性是一份列表，**指定了所有的公共 API 名称**。若没有，则会提供所有的 public 属性（这也是 `import *` 引入的内容）；
- 包目录下的 `__init__.py` 文件也可以通过 `__all__` 来**指定包公开 API 的名称**，其中也可以调用下一级模块的 `__all__` 属性来实现；
- 在实现模块之间的内部 API 时，应当避免使用 `__all__` 来控制 API，命名空间机制足够。

```python
# __init__.py
__all__ = []
from . models import *
__all__ += models.__all__
from . utils import *
__all__ += utils.__all__
```

## 86. 使用模块作用域的代码来配置部署环境

- **在 `__main__` 模块中先指定用于配置环境的全局变量，然后 `import` 主要模块**；
- 在主要模块中，**引入 `__main__` 主模块**（加入全局配置变量），然后将不同配置下的类赋值给统一的变量，以此来实现后续代码的统一。

```python
# dev_main.py
TESTING = True
import db_connection
db = db_connection.Database()

# prod_main.py
TESTING = False
import db_connection
db = db_connection.Database()

# db_connection.py
import __main__
if __main__.TESTING:
    Database = TestingDatabase
else:
    Database = RealDatabase
```

## 87. 为自编的模块定义根异常，以便将调用者与 API 相隔离

- 模块抛出的异常与类和函数一样都是接口的一部分；
- 如果调用者以合理的方式使用 API，那么他们应该会捕获到该模块自己抛出的异常，以此来提醒使用者为此异常添加处理逻辑，而非调用上的错误；
- 如果发现本模块抛出其他类型的异常，则说明代码实现有 bug；
- 使用根异常便于异常继承关系的维护与更新，可以复用父类异常的代码来捕获子类异常。

```python
try:
    weight = my_module.determine_weight(1, -1)
except my_module.InvalidDensityError:
    weight = 0
except my_module.Error as e:
    logging.error('Bug in the calling code: %s', e)
except Exception as e:
    logging.error('Bug in the API code: %s', e)
```

## 88. 用适当的方式打破循环依赖关系

- **`import` 语句的详细机制**（深度优先顺序）：
  - 从 `sys.path` 搜寻待引入的模块；
  - 加载模块的代码，并保证能够正确编译；
  - 创建空对象，添加到 `sys.modules`；
  - 运行模块对象的代码，定义其成员。
- **在创建完模块对象后，就可以使用 `import` 引入，而此时的模块是未定义的（仅执行了第一行引入，无成员）**。在循环依赖中，后一模块引入前一模块时，前一模块还未完成定义，因此在进行后一模块的定义时会出现 `AttributeError` 异常；
- 可能的解决方式：
  - 重构代码，**将共享的数据结构放在依赖树最底层**；
  - 调整引入顺序，先定义共享数据，再引入模块。这与 PEP 8 风格指南不符；
  - 先引入、再配置、最后运行：只在模块中给出函数、类、常量的定义，但不要在引入时真正地运行。所有模块引入后，在主程序中使用单独定义的 `configure` 函数访问其他模块的数据，加载真正的数据内容。问题在于这样划分阶段有时比较困难，且使代码更难懂；
  - 动态引入：在函数运行的时候，在内部引入外部模块。不过 `import` 语句执行开销较大，且可能意外的异常。

## 89. 使用 `warnings` 来重构与移植 API 的使用

- `warnings` 的含义：**提醒其他开发者需要改动代码（由于依赖库代码的变动）**，仅用于合作者之间的信息交流；
- 如果需要修改参数的使用方式，可以使用关键字参数用于过渡，并在没有指定参数时抛出警告；
- `contextlib.redirect_stderr()` 可用于重定向异常输出到给定 IO；
- 警告的信息应该为调用的位置，而非模块内部的位置：**`warnings.warn(stacklevel=...)` 指定警告堆栈的层级**；
- 可以通过设置 `warnings.simplefilter('error'/'ignore')` 来决定是否将警告视为异常抛出。也可以在命令行中通过 `python -W error ...` 指定；
- 生产环境中应当将警告通过 `logging` 模块记录；
- 上下文管理器 `warnings.catch_warnings(record=True)` 可用于在 API 设计时的单元测试中捕捉警告，并与预期比较。

## 90. 使用 `typing` 进行静态代码分析

- `typing` 模块并不提供类型检查的功能，仅仅提供**类型的定义**。静态分析的工具会基于此进行分析（如 IDE 中的插件）；
- `python -m mypy --strict ...` 可以在执行前进行代码检测；
- 支持动态类型的检测：

```python
from typing import Callable, List, TypeVar
Real = TypeVar('Real', int, float)
Func = Callable[[Real, Real], Real]
```

- `Optional[...]` 类型表示可选类型（可以为 None）；
- Python 中的虚数单位使用 `j` 来表示，如 `6+4j`；
- Python 中的类型注释不包括异常的类型，需要使用测试用例来测试异常；
- 可以使用**前向引用**（forward reference）来表示还未定义的类型，也可使用 `from __future__ import annotations` 来让解释器忽视该问题；
- 代码类型注释的使用场景：
    - 简单代码不使用类型注释，在测试时添加一些重要的类型标注；
    - 为重要的 API 接口添加类型注释和警告；
    - 编写复杂代码时避免问题；
    - 便于测试阶段的静态分析。
