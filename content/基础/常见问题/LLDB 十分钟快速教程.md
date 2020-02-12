---
title: "LLDB 快速教程"
date: 2020-02-11T00:10:14+08:00
weight: 1
---

### 概述

macOS 默认使用 LLDB 来进行 C/C++ 程序的调试， LLDB 能够逐行调试程序，使开发者能够了解程序的变量值以及堆栈是如何变化的，一旦学会之后使用起来也比 `printf` 更加方便和简单，赶紧学起来吧。

### LLDB 实现原理

在此之前，请考虑如何实现一个能够监听其他程序（被监听者称为 Client）运行情况的程序（监听者称为 Server）。

1. 第一种方式是 Server 拷贝 Client 的代码来模拟 Client 运行，并且在运行的过程中，Server 通过在模拟过程中使用额外的指令从而能够查看和修改 Client 的运行堆栈和数据信息，其中，Valgrind 就是这样实现的。这种方式的优点是无需预先编译 Client 程序，缺点是因为需要运行额外的指令所以 Server 的运行会比 Client 慢很多（Valgrind 大概会会原程序慢 20-50 倍）。
2. 第二种方式是使用操作系统的 `ptrace` 系统调用，这也是 LLDB 的实现方式。`ptrace` 系统调用可以让 A 进程监听和控制 B 进程的内存和寄存器。`ptrace` 系统调用有以下几个主要功能：
    
    - 捕获 `exec` 系统调用并阻止程序的运行。
    - 查询 CPU 的寄存器来获取当前的指令，数据和栈地址。
    - 监听 clone/fork 事件来判断是否创建新的线程。
    - 读取或者修改 Client 内存变量。

也就是说利用 `ptrace` 系统调用，Client 运行的每一行代码的情况 Server 都能知道。

### 常用指令
这里我们先简单列出来 LLDB 的常见指令，接下来的例子会介绍如何使用（其中括号中的为指令的缩写，例如 `break main` 可以缩写为 `b main`）：

    break (b) - 设置断点，也就是程序暂停的地方
    run (r) - 启动目标程序，如果遇到断点则暂停
    step (s) - 进入下一条指令中的函数内部
    backtrace (bt) - 显示当前的有效函数
    frame (f) - 默认显示当前栈的内容，可以通过 `frame arg` 进入特定的 frame（用作输出本地变量）
    next (n) - 运行当前箭头指向行
    continue (c) - 继续运行程序直到遇到断点。

### 示例

C 标准库中的 `strlen` 函数的作用是找到字符串 s 的长度，例子如下：

    #include <stdio.h>

    size_t strlen(const char *s) {
        const char *sc;

        for (sc = s; *sc != '\0'; ++sc)
            /* nothing */;
        return sc - s;
    }

    int main() {
        // 创建 str 字符串
        char str[] = "Hello World";
        // 调用 strlen 函数，并把值赋予 length
        int length = strlen(str);
        // 在终端打印内容
        printf("The length of str is %d\n", length);
        return 0;
    }

如果你不熟悉 C/C++ 的话，可能不太理解 `strlen` 函数的实现方式，这时候就是 LLDB 大显身手的时候了，使用 LLDB 调试以下程序之前，有几个步骤：

1. 把上面的例子保存为 test.c
2. 在终端运行 `gcc test.c -g -o test` （这里的 -g 参数保证 LLDB 显示的是源代码而不是汇编代码）
3. 终端运行 `lldb test`，这是告诉 LLDB 要调试哪个程序，没有问题的话，终端会输出：

        /path $ lldb test
        (lldb) target create "test"
        Current executable set to 'test' (x86_64).

### 运行程序
这时候 LLDB 已经在监听 test 程序了，test 的一举一动都逃不过 LLDB 的法眼。最基础的命令是 `run`，这条指令会开始运行 test 程序。终端会输出：

    1. Process 9782 launched: '/path/test' (x86_64)
    2. The length of str is 11
    3. Process 9782 exited with status = 0 (0x00000000) 

这里，第一行标示了进程的 ID，第二行是 test 程序的输出，也就是 str 字符串的长度。最后的是程序的返回值，在这里 0 则为正常结束。当然，像这样仅仅有一个输出和返回值对我们调试没有什么帮助。因为程序运行得太快一下子就结束了，我们还没有来得及理解这个程序。LLDB 对于 printf 的优点在于可以逐步调试，我们可以选择一行行地运行程序，然后输出我们需要的堆栈信息以及变量值。让我们重新开始，我们先使用 `Control + C` 退出 LLDB 重新运行 `lldb test` ，然后运行 `break main`，这句指令代表我们在 `main` 函数的开头打上断点，（`break 11` 也能得到相同的结果，这里 11 是 main 的行号）代表让 test 程序在运行到 main 函数的时候暂停，这时候再次运行 `run`, 程序就会在 12 行停止。

    (lldb) break main
    Breakpoint 1: where = test`main + 33 at test.c:12:10, address = 0x0000000100000f01
    (lldb) run
    * thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
        frame #0: 0x0000000100000f01 test`main at test.c:12:10
       9   	}
       10  	
       11  	int main() {
    -> 12  	    char str[] = "Hello World";
       13  	    int length = strlen(str);
       14  	    printf("The length of str is %d\n", length);
       15  	    return 0;
    Target 0: (test) stopped.

那么 `break` 指令是怎么实现的呢？为什么可以让程序在特定的地方暂停呢？简单来说：

    1. `break` 指令 会在参数所在地写入一个无效的地址值，在例子中，则是 main 函数。
    2. 因为地址无效，所以 `test` 程序运行出错，抛出异常，系统会传送 SIGTRAP 信号给 LLDB。
    3. LLDB 这时候可以查看需要的堆栈信息或者变量值。
    4. LLDB 把正确的下一条指令重新写入到 test 程序中。

箭头指向的 12 行是**下一条要执行的指令**，这时候 str 还没进行定义，使用 `print` 指令来验证。

    (lldb) print *str
    (char) $0 = '\0'

要运行 12行 的代码，我们试试 `next` 指令：

    (lldb) next
    * thread #1, queue = 'com.apple.main-thread', stop reason = step over
        frame #0: 0x0000000100000f15 test`main at test.c:13:18
       10  	
       11  	int main() {
       12  	    char str[] = "Hello World";
    -> 13  	    int length = strlen(str);
       14  	    printf("The length of str is %d\n", length);
       15  	    return 0;
       16  	}

再次查看 str 的值：

    (lldb) p *str
    (char) $1 = 'H'

这时候 str 已经被定义了，指向了 'H'，这也是我们预料之中。`frame variable` 用作列出当前所有的变量值。

    (lldb) frame variable
    (char [12]) str = "Hello World"
    (int) length = 0

如果要修改某个变量的值，可以使用 `expr`

    (lldb) expr *str = 'A'
    (char) $2 = 'A'
    // 再次查看
    (lldb) frame variable
    (char [12]) str = "Aello World"
    (int) length = 0

使用 `expr` 之后，str 的值已经变成 "Aello World" 了。下一行要运行的代码是 13 行，这个表达式包含了一个函数调用，当运行 13 行的时候，strlen 函数会被压到 test 程序的栈顶，如下图。

![img]()

使用 `step` 进入函数内部（如果使用 next 的话我们会运行到 14 行，这时候 strlen 函数已经执行完毕了）。

    (lldb) step
    Process 14972 stopped
    * thread #1, queue = 'com.apple.main-thread', stop reason = step in
        frame #0: 0x0000000100000e98 test`strlen(s="Aello World") at test.c:6:15
       3   	size_t strlen(const char *s) {
       4   	    const char *sc;
       5   	
    -> 6   	    for (sc = s; *sc != '\0'; ++sc)
       7   	        /* nothing */;
       8   	    return sc - s;
       9   	}
    Target 0: (test) stopped.

看到箭头指向的是 6 行，我们已经进入到 strlen 函数内部了。使用 `backtrace` 来显示当前的有效函数信息，可以看到我们当前 fram #0 也就是当前在 strlen。

    (lldb) backtrace
    * thread #1, queue = 'com.apple.main-thread', stop reason = step in
      * frame #0: 0x0000000100000e98 test`strlen(s="Aello World") at test.c:6:15
        frame #1: 0x0000000100000f1a test`main at test.c:13:18
        frame #2: 0x00007fff6d7e77fd libdyld.dylib`start + 1

从 6行的代码我们可以看到程序一直在 for 循环中运行，每次运行都会判断 sc 是否已经到达 s 字符串的结尾，如果是则停止 for 循环，strlen 函数最后返回 sc 和 s 的距离。strlen 函数结束后，返回值被赋予到 main 函数的 length 中。继续运行 next 命令，箭头指向 14 行，

    (lldb) n
    Process 15186 stopped
    * thread #1, queue = 'com.apple.main-thread', stop reason = step over
        frame #0: 0x0000000100000f1f test`main at test.c:14:41
       11  	int main() {
       12  	    char str[] = "Hello World";
       13  	    int length = strlen(str);
    -> 14  	    printf("The length of str is %d\n", length);
       15  	    return 0;
       16  	}

这时候 length 的值已经更新了，我们可以通过 `frame variable` 来验证：

    (char [12]) str = "Aello World"
    (int) length = 11

再次使用 `next` 命令，终端输出了 `The length of str is 11`，这也是我们想要的结果。

### 总结

LLDB 的基本用法已经介绍结束了，虽然 LLDB 的命令非常多，不过关键的就是 break, next, step, frame, 这几个，更多的使用例子可以参考官方文档：https://lldb.llvm.org/use/map.html
