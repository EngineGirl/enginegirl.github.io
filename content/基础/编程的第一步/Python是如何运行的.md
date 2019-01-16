---
title: "Python是如何运行的（未完成）"
date: 2019-01-16T00:21:14+08:00
---

世界上最好的编程语言是什么，取决于你用来做什么，不过正确的答案往往是提问者喜欢的编程语言。不过无可否认的是，Python 正慢慢变成全球最受欢迎的语言，没有之一。

[]()

各个编程语言的实现方法各不相同，尽量从设计角度来介绍 Python 的运行原理。这篇文章着重介绍 Python 语言的运行原理和流程，尽量少涉及代码。Python 语言有多种实现方式，Guido van Rossum 在 1989 年趁着家人去旅游写下的第一个版本是用 C 语言实现的（感谢那次旅游），至今最流行的实现也是这个版本，称为 CPython。如果你从来没接触过编程或者 Python 也没关系，它的代码以可读性高而闻名（可能也是他流行的原因）如果你听过欧几里得算法（找出两个数字的最大公因数）这就是欧几里得算法的实现代码：

    def euclid(m, n):
        if n > m:
            m, n = n, m
        r = m % n
        while r:
            m, n = n, r
            r = m % n
        return n


那么计算机是如何理解并且运行 Python 的源代码的呢？CPython 分为编译与解释两个过程，首先，计算机按照顺序从上到下一条条执行指令的，计算机的优点是它能迅速地完成一些重复的工作，例如 1秒钟 进行百万次的加减，赋值。由于计算机不能像人类一样理解过于抽象的内容，例如函数，类（即使可以，也会非常耗时），所以需要把复杂，抽象的源代码简化为机器能够理解，能够快速运行的代码，这个过程就是 Python 的编译过程。

![compiler](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/compiler.png)

#### [tokenizer 过程](#tokenizer-过程)
编译器首先要理解源代码，首先要分析每个字符串的含义，例如 **def** 和 **class** 这类关键字的含义与普通的字符串就不一样。

![tokenizer](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/tokenizer.png)

举一个例子 hello.py

    def say_hello():
        print("Hello, World!")

下面我们使用 Python 自带的 tokenize 模块分析 hello.py，我们可以看到 tokenize 模块把 **"Hello, World"** 当成 **STRING** 类型，而 **def, say\_hello** 等当成了 **NAME** 类型。

    $ python -m tokenize hello.py
    0,0-0,0:            ENCODING       'utf-8'
    1,0-1,3:            NAME           'def'
    1,4-1,13:           NAME           'say_hello'
    1,13-1,14:          OP             '('
    1,14-1,15:          OP             ')'
    1,15-1,16:          OP             ':'
    1,16-1,17:          NEWLINE        '\n'
    2,0-2,4:            INDENT         '    '
    2,4-2,9:            NAME           'print'
    2,9-2,10:           OP             '('
    2,10-2,25:          STRING         '"Hello, World!"'
    2,25-2,26:          OP             ')'
    2,26-2,27:          NEWLINE        '\n'
    3,0-3,1:            NL             '\n'
    4,0-4,0:            DEDENT         ''
    4,0-4,9:            NAME           'say_hello'
    4,9-4,10:           OP             '('
    4,10-4,11:          OP             ')'
    4,11-4,12:          NEWLINE        '\n'
    5,0-5,0:            ENDMARKER      ''



#### parser 过程
理解了源代码包含哪些元素之后，就需要组合它的结构，组合成解释器能够理解的句子，就像把句子的主谓宾放好位置一样，这个过程称为 parser。

![parser](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/parser.png)

#### pyc 文件
经过 tokenizer 和 parser ，编译器把 .py 后缀的源代码转换成了二进制的 bytecode。也就是 \_\_pycache\_\_ 文件夹中的 .pyc 文件。


在解释器眼中的 bytecode 就是一条条指令，我们可以通过 Python 自带的 dis 模块阅读 bytecode 包含的指令。
![interpreter](https://coding.net/u/WindsonYang/p/WindsonYang.coding.me/git/raw/markdown/images/base/how_python/interpreter.png)

#### 解释过程
我们通过 dis 模块获取了解释器所需要的反汇编码

#### 基于栈的虚拟机
栈是计算机中常用的数据结构，栈符合“后进先出”的规则，就像抽屉一样，按照顺序放 A，B，C。那么 C 会在抽屉顶，那么从栈中获取到的第一个元素是 C

![stack]()

Python 使用的是基于栈的虚拟机，（另外还有基于寄存器的虚拟机），Python 会把源代码：
    
    a = 'A'
    b = 'B'
    a + b

转换成类似

    STORE_NAME（把 a 与 b 两个变量的值存在栈中）
    ...
    LOAD_NAME（获取栈顶的元素 a 和 b）
    ...
    BINARY_ADD（把 a 和 b 相加）

这样的字节码，Python 通过 STORE_NAME 先把 a 与 b 两个变量的值存在栈中，LOAD_NAME 获取栈顶的两个元素。



#### 运行过程
运行过程其实就是解释器（包含了基于栈的虚拟机）一句句地执行这些反汇编码，例如把变量压入栈，获取栈顶的变量。



