---
title: Chapter 4 Text versus Bytes
weight: 4
type: docs
---

## 4.1 字符

- Python3 的 `str` 对象中获取的元素时**Unicode 字符**，而非字节序列（Python2）；
- Unicode 标准中的**码位**和**编码**：
    - 码位：字符的标识，用 U+xxxx（4 位到 6 位十六进制数字）表示（表示为\\uxxxx）；
    - 字符序列：实际的表示，用两个十六进制数字表示一个字符（如\\x41）；
    - 编码：字符的具体表述，在**码位和字节序列间转换**的算法，如 UTF-8 编码（A 编码为\\x41）、UTF-16LE 编码（A 编码为\\x41\\x00）等（上述编码得到的结果为**字节序列**）。
    - 将码位转为字符序列称为编码（`.encode()`，**将人类可读的 `str` 编码为 `bytes`**。注意 `str` 中存储的是 Unicode 字符，也即码位），反向称为解码（`.decode()`，将 `bytes` 解码为 `str`）；
- `bytes` 字面量以 `b` 开头。

```python
s = 'café'
len(s)                 # 4
b = s.encode('utf8')   # b'caf\xc3\xa9'
len(b)                 # 5
b.decode('utf8')       # 'café'
```

## 4.2 字节

### 概述

- 内置的基本**二进制序列类型**：不可变的 `bytes` 和可变的 `bytearray`；
- **元素均是 0-255 的整数（下标读取）**，切片仍然是同一类型的二进制序列；
- 显示方式：
    - 可打印的 ASCII：字符本身；
    - 空白符：使用转义序列；
    - 其他字节：使用十六进制转义序列 `\x00`。

### 方法

- 可以使用大部分 `str` 类的方法来处理 `bytes` 和 `bytearray`（除了格式化方法和 Unicode 相关）；
- 特殊的**类方法**：`fromhex`，解析十六进制数字对变成字符，如 `bytes.fromhex('31 4B CE A9')`；
- 构造方式：
    - 对 `str` 调用编码 `.encode()` 方法构造 `bytes`；
    - 使用 `bytes/bytearray(string, encoding=xxx)` 函数构造；
    - 接受包含 0-255 之间数值的可迭代对象；
    - 一个实现了**缓冲协议的对象**（`bytes`、`bytearray`、`memoryview`、`array.array`）等，直接复制其中的字节序列到新的二进制序列中。这种方式可能涉及**强制类型转换**；

### 结构体与内存视图

- 结构体模块 `struct`：将打包的字节序列转换为不同类型字段组成的**元组**，或反向打包；
- 拆包 `struct.unpack(format, data)`：**结构体的格式**`format` 用来指定拆包的形式，如 `<3s3sHH` 中 `<` 表示小字节序，`3s` 表示 3 字节序列，`H` 表示 16 位二进制整数。

```python
import struct

with open('filter.gif', 'rb') as fp:
    img = memoryview(fp.read())

header = img[:10]
print(bytes(header))                       # b'GIF89a+\x02\xe6\x00'
print(struct.unpack('<3s3sHH', header))    # (b'GIF', b'89a', 555, 230)
```

## 4.3 基本的编解码器

- 部分编码不能表示所有 Unicode 字符，UTF 编码的设计目的就是处理每一个 Unicode 码位；
- UTF-16LE 中的 LE 表示 **Little Endian（小字节序）**。

## 4.4 编解码问题

### 异常处理

- `UnicodeEncodeError`：如果目标编码中没有定义该字符，就会抛出该异常。除非**传入 `errors` 参数，对错误进行特殊处理**。默认为 `strict`，可以选择：
    - `ignore` 跳过无法编码的字符；
    - `replace` 把无法编码的字符替换为 `'?'` 字符；
    - `xmlcharrefreplace` 把无法编码的字符替换成 XML 实体（格式：`&#xxx;`）；
    - 可以**扩展**其他的错误处理方式，使用 `codecs.register_error` 函数；
- **utf-? 可以处理任何字符串**。
- `UnicodeDecodeError`：如果字节不包含有效的 ASCII 字符，则抛出该异常。同样可设置 `errors`；
    - 如果使用 `replace` 进行错误处理，则会替换为“替换字符”（Replacement Charater）`�`；
    - 如果使用错误的编码，解码过程可能**不会出现错误，而得到乱码字符**（鬼符）。

### 加载 Python 模块时的编码问题

- Python3 默认**使用 UTF-8 编码源码**，Python2 为 ASCII；
- 如果需要在源码中包括 UTF-8 以外的数据，可以在开头标注 `# coding: xxx` 表示**编码方式**。
- 注意：要使用不同的编码**保存**（设置文件编码格式），这样才需要标注编码方式。否则，如果**使用默认的 UTF-8 保存，会自动将其转换为 UTF-8**，这样就不需要特殊编码。因此，要避免编码问题，将其保存为 UTF-8 编码格式即可。

### 判断字节序列的编码

- 经验：`b'\x00` 出现只可能为 16/32 位编码；`b'\x20\x00` 很可能为 UTF-16LE 中的空格字符；
- Python 库 `chardet` 通过试探和分析来识别所支持的 30 种编码，并提供了命令行工具。

### BOM：字节序标记

- 使用 UTF-16 编码的文本要在开头加上**不可见字符** `ZERO WIDTH NO-BREAK SPACE`（U+FEFF），**在小字节序中对应字符编码 `b'\xff\xfe'`**，在大字节序中相反，可以以此判断。UTF-16 编解码器可以通过 BOM 判断字节序，并将其过滤掉，提供真正的文本内容；
- 如果使用 UTF-16 的变种 LE、BE 显式指名字节序，则**不会生成 BOM**；
- 根据标准，如果文件使用 UTF-16 编码且没有 BOM，则应该假定大字节序。但也有例外；
- UTF-8 中也可能存在 BOM，U+FEFF 字符为为三字节序列 `b'\xef\xbb\xbf'`。

## 4.5 处理文本文件

### 文件编解码

- **Unicode 三明治**：解码输入的字节序列，**只处理文本**，最后编码输出；
- 打开文件时若不指定编解码方式，则会使用区域设置中的默认编码。因此**不能依赖默认编码**；
- 使用文本、二进制方式打开文件时，得到的文件描述符对象分别为 `TextIOWrapper` 和 `BufferedReader`。前者调用 `read()` 方法得到字符串，后者得到字符序列 `bytes`。

### 默认编码

- **小技巧：**
    - `eval` 函数能够将**字符串翻译为语句**，而不仅仅是字面值的转换；
    - `str.rjust(width)` 可以用于**右对齐显示**；
- `locale.getpreferredencoding()` 返回的编码是打开文件的默认编码，也是重定向标准输出的默认编码。但它在不同系统中设置方式不同，返回值只是猜测的编码。

## 4.6 规范化 Unicode 字符串

### 规范化 `unicodedata.normalize`

- 规范化的原因：Unicode 中包含组合字符（变音符号、附加符号），它们可能构成“**标准等价物**”（canonical equivalent），看上去一样的字符串却不相等；
- `unicodedata.normalize(mode, string)` 可以用来规范化字符串，可用的模式：
    - NFC：最少的码位（看成**整体**），是**W3C 推荐的规范化形式**。有些看上去完全相同的单字符会被规范为另一个（如Ω），因此需要规范化；
    - NFD：**分解**成基字符和单独的组合字符；
    - NFKC、NFKD：（K-兼容性）严格的规范化形式，将“**兼容字符**”（用来兼容现有标准而添加的重复字符）替换为若干个字符，但会导致**格式损失**，如²、½等。因此只在**搜索、索引**等特殊情况中使用。

### 大小写折叠

- `s.casefold()` 与 `s.lower()` 一样，把所有文本变成小写。但有少量的字符是例外，如微符号、德语中的 Eszett 等。

### 规范化文本匹配

```python
"""  
Utility functions for normalized Unicode string comparison.  
  
Using Normal Form C, case sensitive:  
  
    >>> s1 = 'café'  
    >>> s2 = 'cafe\u0301'  
    >>> s1 == s2  
    False  
    >>> nfc_equal(s1, s2)  
    True  
    >>> nfc_equal('A', 'a')  
    False  
  
Using Normal Form C with case folding:  
  
    >>> s3 = 'Straße'  
    >>> s4 = 'strasse'  
    >>> s3 == s4  
    False  
    >>> nfc_equal(s3, s4)  
    False  
    >>> fold_equal(s3, s4)  
    True  
    >>> fold_equal(s1, s2)  
    True  
    >>> fold_equal('A', 'a')  
    True  
  
"""  
  
from unicodedata import normalize  
  
def nfc_equal(str1, str2):  
    return normalize('NFC', str1) == normalize('NFC', str2)  
  
def fold_equal(str1, str2):  
    return (normalize('NFC', str1).casefold() == normalize('NFC', str2).casefold())
```

### 去掉变音符号

- 先使用 NFD 规范化分解，之后过滤掉所有**组合符号**（用 `unicodedata.combining(c)` 可以判断组合符号）：

```python
import unicodedata  
import string  
  
  
def shave_marks(txt):  
    """Remove all diacritic marks"""  
    norm_txt = unicodedata.normalize('NFD', txt)  # <1>  
    shaved = ''.join(c for c in norm_txt  
                     if not unicodedata.combining(c))  # <2>  
    return unicodedata.normalize('NFC', shaved)  # <3>
```

- 对非拉丁字符的修改没有意义，无法将其改为 ASCII 字符。因此需要对前一**基字符**判断：

```python
def shave_marks_latin(txt):  
    """Remove all diacritic marks from Latin base characters"""  
    norm_txt = unicodedata.normalize('NFD', txt)  # <1>  
    latin_base = False  
    keepers = []  
    for c in norm_txt:  
        if unicodedata.combining(c) and latin_base:   # <2>  
            continue  # ignore diacritic on Latin base char  
        keepers.append(c)                             # <3>  
        # if it isn't combining char, it's a new base char  
        if not unicodedata.combining(c):              # <4>  
            latin_base = c in string.ascii_letters  
    shaved = ''.join(keepers)  
    return unicodedata.normalize('NFC', shaved)   # <5>
```

- 还可以将常见特殊符号替换为对等的字符/字符串：

```python
single_map = str.maketrans("""‚ƒ„†ˆ‹‘’“”•–—˜›""",  # <1>  
                           """'f"*^<''""---~>""")  
  
multi_map = str.maketrans({  # <2>  
    '€': '<euro>',  
    '…': '...',  
    'Œ': 'OE',  
    '™': '(TM)',  
    'œ': 'oe',  
    '‰': '<per mille>',  
    '‡': '**',  
})  
  
multi_map.update(single_map)  # <3> 合并上面的两个映射表 
  
  
def dewinize(txt):  
    """Replace Win1252 symbols with ASCII chars or sequences"""  
    return txt.translate(multi_map)  # <4>  
  
  
def asciize(txt):  
    no_marks = shave_marks_latin(dewinize(txt))     # <5>  
    no_marks = no_marks.replace('ß', 'ss')          # <6>  
    return unicodedata.normalize('NFKC', no_marks)  # <7>
```

- 上述的规范化操作超出了标准的范围，很可能会改变原意，需要根据应用场景确定。

## 4.7 Unicode 文本排序

- 问题：在众多语言中，重音符号和下加符对排序没有影响，不应当直接按**码位**排序（默认）；
- 非 ASCII 文本的标准排序方式：`key=locale.strxfrm` 将字符串转换为**所在区域进行比较的形式**。因此需要预先使用 `locale.setlocale(locale.LC_COLLATE, <<your_locale>>)` 设置合适的**区域**。不过这种方法由很多问题：区域设置是全局的，操作系统必须支持区域设置，需要区域名称，操作系统正确实现了该区域；
- **PyUCA**：**Unicode 排序算法**（Unicode Collation Algorithm）的纯 Python 实现：
    - `coll = pyuca.Collator()` 构造排序工具；
    - `key=coll.sort_key` 指定排序关键码函数；
    - 可以自行定制排序方式，传入构造函数。默认使用 Unicode 6.3.0 的 Default Unicode Collation Element Table。

## 4.8 Unicode 数据库

- Unicode 标准提供了完整的数据库，记录了字符是否**可打印**、**是字母**、**是数字**等性质。字符串的若干方法就是基于此实现的；
- 小技巧：
    - `str.center(width)` **居中对齐**；
    - `%04x` 表示**十六进制、4 位、空缺填 0**；
    - 可以使用 `r = re.compile(expr)` 一次性设置匹配的规则，随后用 `r.match()` 即可匹配；
    - `unicodedata.numeric(char)` 能够**将字符转化为其表示的数字**。

## 4.9 支持字符串和字节序列的双模式 API

> 双模式 API：接受字符串或字符序列为参数，**根据类型展现不同的行为**。

### 正则表达式

- 使用**字节序列**构建：只能匹配 ASCII 字符（**严格对应**）；
- 使用**字符串**构建：可以匹配其他 Unicode 数字或字母； ^34d285
- 字节序列只能用字节序列正则表达式搜索（**类型必须对应**）；
- 可以使用 `re.ASCII` 标志指定只能匹配 ASCII 字符；
- 小技巧：
    - 正则表达式中 `\d` 表示**数字**，`\w` 表示**数字或字母**；
    - 可以使用**括号**的方式**拼接**两行字符串：

```python
a = ("2321414"
"12312313")

print(a)  # 232141412312313
```

### `os`中的函数

- `os.listdir()`返回路径中的**文件名列表**。参数是字节序列，则返回的文件名也是字节序列；
- 手动处理编解码：`os.fsencode(filename)`、`os.fsdecode(filename)`。它们的错误处理方式为`surrogateescape`（Linux）或`strict`（Windows）：
	- `surrogateescape`会将无法解码的字节替换成U+DC00到U+DCFF之间的码位（Low Surrogate Area），这些码位是保留的，没有分配字符。用同样的模式编码可以还原。

