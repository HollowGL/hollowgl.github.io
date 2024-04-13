---
title: typing在3.9中的变化
toc: false
excerpt: Python3.9支持直接使用tuple, list等进行类型注解
date: 2024-04-14 00:00:38
updated: 2024-04-14 00:00:38
cover:
thumbnail:
categories:
tags:
---


<!-- more -->

## 初识typing

Python在3.5版本加入typing模块，用于支持类型提示。平常几十行的小项目可能不会用到，但是在大型项目中，类型提示可以显著提高代码的可读性和可维护性。
```python
def greeting(name: str) -> str:
    return 'Hello ' + name
```
不过，Python 运行时不强制执行函数和变量类型注解，但这些注解可用于类型检查器、IDE、静态检查器等第三方工具。比如vscode就不会对其进行检查，因此需要使用mypy等工具。

```shell
pip install mypy
```

命令行执行 `mypy <filename>.py` 即可进行检查
```shell shell
⮞  cat .\demo.py
def greeting(name: str) -> str:
    return 1

⮞  mypy .\demo.py
demo.py:2: error: Incompatible return value type (got "int", expected "str")  [return-value]
Found 1 error in 1 file (checked 1 source file)
```




## typing的变化

typing的类型注释有很多种，比如`List`、`Dict`、`Tuple`、`Union`、`Optional`等等，这些类型注释可以组合使用，形成更复杂的类型注释。但是这些类型需要从typing模块导入：

```python
from typing import Tuple

def foo(a: int) -> Tuple[int]:
    return (a,)
```

注意这里的Tuple的T是大写的，而在Python3.9以后，可以直接使用内置的`tuple`、`list`、`dict`等类型，不需要导入typing模块（但是依然支持从typing导入）。

```python
def foo(a: int) -> tuple[int]:
    return (a,)
```

不过，实际上3.9以前也能直接使用小写的 `tuple`，但是无法指定其类型：
```python
def foo(a: int) -> tuple:
    return (a,)
```
这种写法在使用mypy检查时并不会报错，但是一旦使用 `tuple[int]`，无需mypy出手，IDE就会指出错误。

这个问题在python3.7以后可以通过加入 `from __future__ import annotations` 来解决，这样就可以直接使用内置类型了。参见[PEP 563](https://peps.python.org/pep-0563/)。

```python   
from __future__ import annotations

def foo(a: int) -> tuple[str]:
    return str(a),
```

不过这种方式可能用得少，毕竟`Union`、`Optional`、`Callable`等类型还是需要从typing导入的，以上只是typing的冰山一角。