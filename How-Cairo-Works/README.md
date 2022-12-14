#  How Cairo Works
“How Cairo Works” 从 **Cairo** 的低级类汇编版本开始，解释了 **Cairo** 编译器的语法糖机制，它将 **Cairo** 变成了一种类似高级的语言。如果你打算立即上手写代码，可以先参考 “Hello, Cairo” 教程。

- [**Cairo** 介绍](Introduction-to-Cairo.md)
  - [字段](Introduction-to-Cairo.md#field)
  - [非确定性计算](Introduction-to-Cairo.md#compute)
  - [内存模型](Introduction-to-Cairo.md#memory-model)
  - [寄存器](Introduction-to-Cairo.md#register)
  - [基本指令](Introduction-to-Cairo.md#basic-instruction)
  - [连续内存](Introduction-to-Cairo.md#continues-memory)
- [Debug 用到的一些flags](Debugging-related-flags.md)
  - [--print_info](Debugging-related-flags.md#print-info)
  - [--print_memory](Debugging-related-flags.md#print-memory)
  - [--steps](Debugging-related-flags.md#steps)
  - [--no_end](Debugging-related-flags.md#no-end)
  - [--debug_error](Debugging-related-flags.md#debug-error)
  - [--profile_output profile.pb.gz](Debugging-related-flags.md#more)
- [程序计数器 (pc)](The-Program-Counter.md)
  - [程序计数器和跳转](The-Program-Counter.md#counter-and-jump)
  - [条件跳转](The-Program-Counter.md#condition-jump)
- [常量和引用](Consts.md)
  - [常量](Consts.md#const)
  - [短字符串文字](Consts.md#Short-string-literals)
  - [引用](Consts.md#references)
  - [断言和复合表达式](Consts.md#assert)
  - [撤销引用](Consts.md#revoked-references)
  - [类型引用](Consts.md#type-reference)
  - [类型转换](Consts.md#cast)
  - [临时变量](Consts.md#tempvar)
  - [局部变量](Consts.md#local-variables)
  - [类型化的局部变量](Consts.md#typed-local-vars)
  - [引用重绑定](Consts.md#Reference-rebinding)
  - [元组](Consts.md#tuples)
  - [数组](Consts.md#array)
- [函数](Functions.md)
  - [介绍](Functions.md#introduction)
  - [fp寄存器](Functions.md#fp)
  - [深入理解](Functions.md#under-the-hood)
  - [访问寄存器的值](Functions.md#register-value)
  - [函数参数和返回值](Functions.md#function-params-and-returns-value)
  - [函数参数](Functions.md#function_params)
  - [尾递归](Functions.md#recursive)
  - [返回元组](Functions.md#return-tuple)
- [类型](Types.md)
  - [元组](Types.md#tuple)
  - [用户定义的类型别名](Types.md#alias)
- [对象创建](Object-Allocation.md)
  - [alloc()](Object-Allocation.md#alloc)
  - [new 操作符](Object-Allocation.md#new)
- [作用域参数](Scope-Attributes.md)
- [导入包](Imports.md)
  - [导入搜索路径](Imports.md#search-path)
- [提示](Hints.md)
  - [介绍](Hints.md#introduction)
- [程序输入和输出](Program-Input-Output.md)
 - [程序输入](Program-Input-Output.md#program-input)
 - [程序输出](Program-Input-Output.md#program-output)
- [段](Segments.md)
  - [基本原理](Segments.md#fundamental)
  - [可重定位值](Segments.md#relocatable-values)
  - [用例](Segments.md#use-cases)
  - [内置段](Segments.md#internal-segment)
- [非确定性跳转](Nondeterministic-Jumps.md)
- 内置函数和隐式参数
  - 介绍
  - 隐式参数
  - 调用带隐式参数的参数
  - Revoked implicit arguments
  - 布局
  - %builtins 指令
  - 范围检查
- [字面值](Define-Word.md)