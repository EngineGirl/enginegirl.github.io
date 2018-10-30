---
title: "Python是如何运行的（未完成）"
date: 2018-10-29T00:10:14+08:00
---

这篇文章着重分析 Python 语言的运行原理和流程，希望给从没接触过编程或者 Python 初学者一个简单的了解。各位想了解更深层的实现原理，推荐阅读，[10 hours]()和[Python源码剖析]()。Python 语言有多种实现方式，最流行的是 CPython，用 C 语言实现的，CPython 包含一个解释器和编译器。那么计算机是如何运行 Python 的源代码的呢？

#### 编译过程
一般我们把 Python 称为解释性语言，但是大部分的解释性语言为了优化速度都会增加编译过程。编译器首先经过 tokenizer 分析代码的主要部分，就像获取句子的主谓宾一样。

![tokenizer](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/tokenizer.png)

然后经过 parser 分析代码的结构，就像使用主谓宾组合句子一样：

和 parse 之后把源代码转换成 bytecode
![parser](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/parser.png)

Python 的编译器把 .py 后缀的源代码转换成 bytecode 也就是我们在 \_\_pycache\_\_ 文件夹中出现的 .pyc 文件。这个文件是二进制也就是 0 和 1 组成的，不能直接阅读，我们可以通过 Python 自带的 dis 模块阅读 bytecode 包含的指令。这就到解析过程

#### 解释过程
我们通过 dis 模块获取了解释器所需要的反汇编码

![interpreter](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/interpreter.png)

#### 运行过程
运行过程其实就是解释器（包含了基于栈的虚拟机）一句句地执行这些反汇编码，例如把变量压入栈，获取栈顶的变量。


