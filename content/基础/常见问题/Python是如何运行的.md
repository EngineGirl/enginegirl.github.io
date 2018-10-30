---
title: "Python是如何运行的（未完成）"
date: 2018-10-29T00:10:14+08:00
---

这篇文章着重分析 Python 语言的运行原理和流程，希望给从没接触过编程或者 Python 初学者一个简单的了解。各位想了解更深层的实现原理，推荐阅读，[10 hours]()和[Python源码剖析]()。Python 语言有多种实现方式，最流行的是 CPython，用 C 语言实现的，CPython 包含一个解释器和编译器。那么计算机是如何运行 Python 的源代码的呢？

#### 编译过程
一般我们会把 Python 称为解释性语言，不过大部分的解释性语言为了优化速度都会增加编译过程。编译过程同一般的编译性语言如 C 语言，Go 语言类似，主要包括 tokenizer 过程和 parser 过程。

##### tokenizer 过程
编译器首先经过 tokenizer 分析代码，就像获取文章每个句子的主谓宾成分一样。

![tokenizer](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/tokenizer.png)

##### parser 过程
获取了主要成分之后，经过 parser 分析代码的结构，并且组合成解释器能够理解的句子：

![parser](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/parser.png)

##### pyc 文件
经过 tokenizer 和 parser ，编译器把 .py 后缀的源代码转换成了二进制的 bytecode。也就是 \_\_pycache\_\_ 文件夹中的 .pyc 文件。

![compiler](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/compiler.png)

在解释器眼中的 bytecode 就是一条条指令，我们可以通过 Python 自带的 dis 模块阅读 bytecode 包含的指令。
![interpreter](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/interpreter.png)

#### 解释过程
我们通过 dis 模块获取了解释器所需要的反汇编码


#### 运行过程
运行过程其实就是解释器（包含了基于栈的虚拟机）一句句地执行这些反汇编码，例如把变量压入栈，获取栈顶的变量。


