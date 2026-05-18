---
title: Chapter 6 Design Patterns with First-Class Functions
weight: 6
type: docs
---

> 使用的程序设计语言决定了哪些设计模式可用。

## 6.1 重构“策略”模式

### 经典“策略”模式

- **上下文**：不同环节算法之间的衔接部分，它把计算委托给**实现不同算法的可互换组件**（不同的具体策略），它提供服务。在书中的电商实例中，上下文为订单类，接受订单信息，将打折的具体算法交给策略来实现；
- **策略**：多态性的体现，实现不同算法的组件的**共同接口**，是**具体策略的抽象基类**；
- **具体策略**：**策略的具体子类**，实现了其中的抽象方法，适配不同的算法；
- 技巧：**抽象基类**的定义中，类需要继承 `abc.ABC`，**抽象方法**前加上装饰器 `@abstractmethod`，不需要写函数体（Python 3.4 及以后）。

```python
from abc import ABC, abstractmethod  
from collections import namedtuple  
  
Customer = namedtuple('Customer', 'name fidelity')  
  
  
class LineItem:  
  
    def __init__(self, product, quantity, price):  
        self.product = product  
        self.quantity = quantity  
        self.price = price  
  
    def total(self):  
        return self.price * self.quantity  
  
  
class Order:  # the Context  
  
    def __init__(self, customer, cart, promotion=None):  
        self.customer = customer  
        self.cart = list(cart)  
        self.promotion = promotion  
  
    def total(self):  
        if not hasattr(self, '__total'):  
            self.__total = sum(item.total() for item in self.cart)  
        return self.__total  
  
    def due(self):  
        if self.promotion is None:  
            discount = 0  
        else:  
            discount = self.promotion.discount(self)  
        return self.total() - discount  
  
    def __repr__(self):  
        fmt = '<Order total: {:.2f} due: {:.2f}>'  
        return fmt.format(self.total(), self.due())  
  
  
class Promotion(ABC):  # the Strategy: an Abstract Base Class  
  
    @abstractmethod  
    def discount(self, order):  
        """Return discount as a positive dollar amount"""  
  
  
class FidelityPromo(Promotion):  # first Concrete Strategy  
    """5% discount for customers with 1000 or more fidelity points"""  
  
    def discount(self, order):  
        return order.total() * .05 if order.customer.fidelity >= 1000 else 0  
  
  
class BulkItemPromo(Promotion):  # second Concrete Strategy  
    """10% discount for each LineItem with 20 or more units"""  
  
    def discount(self, order):  
        discount = 0  
        for item in order.cart:  
            if item.quantity >= 20:  
                discount += item.total() * .1  
        return discount  
  
  
class LargeOrderPromo(Promotion):  # third Concrete Strategy  
    """7% discount for orders with 10 or more distinct items"""  
  
    def discount(self, order):  
        distinct_items = {item.product for item in order.cart}  
        if len(distinct_items) >= 10:  
            return order.total() * .07  
        return 0
```

### 使用函数实现“策略”模式

- 由于策略模式中的具体策略都只包括一个方法，可以考虑把具体策略换成函数来实现。这种实现的关键原因在于，**Python 中函数都是对象，因此不再需要自定义各策略类，且不需实例化**；
- 在策略模式中，为了避免使用相同策略时不断新建具体策略对象，需要考虑**策略对象的共享——享元**。因此这样实现的优势在于：函数对象只会在编译时创建一次，实现了策略对象的共享。

```python
from collections import namedtuple  
  
Customer = namedtuple('Customer', 'name fidelity')  
  
  
class LineItem:  
  
    def __init__(self, product, quantity, price):  
        self.product = product  
        self.quantity = quantity  
        self.price = price  
  
    def total(self):  
        return self.price * self.quantity  
  
  
class Order:  # the Context  
  
    def __init__(self, customer, cart, promotion=None):  
        self.customer = customer  
        self.cart = list(cart)  
        self.promotion = promotion  
  
    def total(self):  
        if not hasattr(self, '__total'):  
            self.__total = sum(item.total() for item in self.cart)  
        return self.__total  
  
    def due(self):  
        if self.promotion is None:  
            discount = 0  
        else:  
            discount = self.promotion(self)  # <1>  
        return self.total() - discount  
  
    def __repr__(self):  
        fmt = '<Order total: {:.2f} due: {:.2f}>'  
        return fmt.format(self.total(), self.due())  
  
# <2>  
  
def fidelity_promo(order):  # <3>  
    """5% discount for customers with 1000 or more fidelity points"""  
    return order.total() * .05 if order.customer.fidelity >= 1000 else 0  
  
  
def bulk_item_promo(order):  
    """10% discount for each LineItem with 20 or more units"""  
    discount = 0  
    for item in order.cart:  
        if item.quantity >= 20:  
            discount += item.total() * .1  
    return discount  
  
  
def large_order_promo(order):  
    """7% discount for orders with 10 or more distinct items"""  
    distinct_items = {item.product for item in order.cart}  
    if len(distinct_items) >= 10:  
        return order.total() * .07  
    return 0
```

### 选择最佳策略

- 充分利用函数的对象性质，将其**存储在数据结构（如列表）中**，从而借助归约函数（如 `max`）和生成器表达式求解最优解；
- 在修改策略时，需要将新的策略函数添加到列表中。可以使用下一节的主动查找方法自动找到全部策略。

### 找出模块中的全部策略

- 内置函数 `globals()` 返回当前**全局符号表**的字典，这是针对**当前模块的**（模块定义的范围）；
- 将策略函数实现在**独立的模块**中，使用 `inspect.getmembers(module_name, inspect.isfunction)` 获取函数。这里 `getmembers` 能够返回模块中的、判断条件为真的**符号表字典**，判断条件（可选参数） `isfunction` 需要是一个**布尔值函数**，这里能够判断是否为函数。

> 该实例的更显式的解决方案是使用装饰器，见**第 7 章**。

## 6.2 “命令”模式

- **“命令”模式**：**解耦调用者和接受者**，在两者之间放一个 `Command` 对象（`Command` 是抽象基类，操作的实例对象应当为子类对象），调用者选择一个 `Command` 命令，调用它的`execute()`方法执行即可，**无需了解接受者的接口**以及`Command`的具体实现；
- 函数方式实现：用不同的命令函数来表示`command.execute`方法，只需要保存函数对象即可。
