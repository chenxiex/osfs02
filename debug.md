# 使用 qemu+gdb 调试
本仓库的配套教科书中介绍了使用 `bochs` 进行调试的方法。本文介绍如何使用 `qemu` + `gdb` 进行调试，并给出书中指令的对应操作。

# 启动调试
在[https://github.com/chenxiex/osfs01](https://github.com/chenxiex/osfs01)中，已经能够启动qemu虚拟机。如果你查看了Makefile，会发现启动qemu虚拟机的命令为：
```bash
qemu-system-i386 -fda a.img
```
为了启动调试，我们需要稍微修改这个命令。你不必修改Makefile文件，可以直接在控制台运行：
```bash
qemu-system-i386 -fda a.img -s -S
```
其中，`-s` 参数是在1234端口开启 `gdb` 服务器，`-S` 在启动后暂停以等待 `gdb` 连接。
之后，启动 `gdb`
```bash
gdb
```
你应该看到 `(gdb)` 终端。如果 `gdb` 打印了帮助信息，按回车直到出现 `(gdb)` 字样。之后连接到 `qemu`
```bash
(gdb) target remote localhost:1234
```
此时 `gdb` 已经连接到 `qemu` 了。你可以进行一些简单的调试操作。比如，以下操作在 `0x7c00` 位置设置断点，并执行到该断点位置，之后打印当前指令，并继续执行直到退出。如果顺利，你能看到 `0x7c00` 位置的第一条 `mov` 指令，并能在 `qemu` 中看到输出，就像上一节那样。
```bash
(gdb) hbreak *0x7c00
(gdb) c
(gdb) x /i $pc
(gdb) c
```
关闭 `qemu` 窗口即可退出调试。

# 常见 bochs 指令和 gdb 指令对照
下面给出书中提到的 `bochs` 指令和对应 `gdb` 指令的对照表。可能不全，善用搜索！
| 行为 | bochs 指令| gdb 指令| gdb 举例|
|---|---|---|---|
|在某物理地址设置断点|b addr|hbreak *addr|hbreak *0x7c00|
|显示当前所有断点信息|info break|info break|info break|
|继续执行直到断点|c|c|c|
|单步执行|s|stepi|stepi|
|单步执行（遇到函数跳过）|n|nexti|nexti|
|查看寄存器信息|info cpu; r; fp; sreg; creg|info registers; print $reg|info registers; print $eax|
|反汇编执行的每一条指令|trace-on|display /i $pc|display /i $pc|

关于查看堆栈、内存地址内容、当前指令内容，在gdb中都用 `x` 指令来实现。`x` 指令的用法和 `bochs` 中类似指令稍有差别，不好对照，建议直接查询希望进行的操作对应的 `x` 指令。你也可以用
```
(gdb) help x
```
查询 `x` 指令语法。
