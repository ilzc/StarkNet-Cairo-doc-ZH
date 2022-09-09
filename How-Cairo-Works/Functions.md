# 函数
## 介绍

函数是一个可以重复调用的代码单元，通常会包括输入参数和返回值。

函数是一个可重用的代码单元，它接收参数并返回一个值。 为了在 Cairo 中实现这一点，我们引入了两个低级指令：call addr 和 ret。 此外，Cairo 编译器支持这些指令的高级语法：分别为 foo(...) 和 return (...)。

函数的声明方式如下：

```
func function_name() {
    // 在这编写代码
    return ();
}
```

func function_name() { 和 } 行不会被转换为 Cairo 指令——它们只是被编译器用来命名函数并创建相应的作用域。

要调用该函数，只需使用 call 指令：

```
call function_name;
```

或高级语法：

```
function_name();
```

`call` 的完整语法类似于 `jmp：`可以调用标签（函数也被认为是标签），并进行相对或绝对调用（`call rel/abs` ...）。

## fp寄存器
当函数启动时，帧指针寄存器 (fp) 被初始化为 `ap` 的当前值。 在函数的整个范围内（不包括内部函数调用）， `fp` 的值保持不变。 特别是，当函数 foo 调用内部函数 bar 时，fp 的值会在 bar 启动时更改，但在 bar 结束时恢复。

这个想法是，当调用内部函数时，ap 可能会以未知的方式发生变化，因此在此之后它不能可靠地用于访问原始函数的局部变量和参数。 因此，fp 作为访问这些值的锚点，记录了初始位置。

### 练习
编译并运行以下代码（至少 10 个步骤）。 使用 `--print_memory` 和 `--relocate_prints` 进行 `cairo-run`。

```
func main() {
    call foo;
    call foo;
    call foo;

    ret;
}

func foo() {
    [ap] = 1000, ap++;
    ret;
}
```

我们将在下面的例子中分析这个程序的内存。 现在，试着想一想当 cpu 到达 ret 指令时会发生什么（寄存器 ap、fp、pc 中的哪一个在 ret 被执行时应该改变，具体是什么值？）。

## 深入理解

`call addr` 大致相当于下面的一组指令：

```
// 存储当前 fp 的值，一旦调用函数结束使用 ret 将恢复该值。
[ap] <-- fp

// 存储下一条指令的地址，以便在被调用函数结束后运行。 这将在调用 ret 时分配给 pc。
[ap + 1] <-- return_pc

// 将 ap 增加 2，因为共有两次写入。
ap += 2;

// 将 fp 更新为新的 ap，因此它指向被调用函数范围内新帧的开始。
fp <-- ap

jmp addr;

```

`ret` 大致相当于以下指令集：

```
// 跳转到 return_pc（存储在堆栈中）。
jmp [fp - 1];

// 恢复前一个 fp 的值。
fp <-- [fp - 2]
```

我们可以这样总结：

`call` 将当前帧指针和返回地址“推送”到（虚拟）堆栈对（fp，return_pc）并跳转到给定地址。

`ret` “弹出”前一个 `fp` 并跳转到在调用期间推送的 `return_pc`。

在调用指令之后，内存如下所示：

```
|      ...     |
+--------------+
| old_fp       |
+--------------+
| return_pc    |
+--------------+
|              | <-- ap, fp
+--------------+
|      ...     |
```

### 例子
运行上面的程序后查看内存值：
```
Address  Value
13       13
14       3
15       1000
16       13
17       5
18       1000
19       13
20       7
21       1000
```

main 启动时，`fp` 的值为 13，程序调用 `foo` 的第一次调用，写入 `fp` 的当前值 (13) 和程序计数器的值返回 (3)。 foo 将 1000 写入内存。 `ret` 指令将 `fp` 的值恢复为 13，然后跳转到 `pc = 3`。然后再次调用 foo。 确保您了解其余的内存值。

### 更多例子
使用上一节中给出的信息，编写一段代码，在执行时将 `ap`、`fp` 和 `pc` 的当前值放入内存中（例如，将 `ap` 写入 [ap]，将 `fp` 写入 `[ap + 1]` 和 `pc` 进入 `[ap + 2]`)。

## 访问寄存器的值
Cairo 的标准库有两个函数可以检索三个寄存器的值（实际上，它们的实现与上一个练习的解决方案类似）。 您可以按如下方式使用它们：

```
from starkware.cairo.common.registers import get_ap
from starkware.cairo.common.registers import get_fp_and_pc

let get_ap_res = get_ap();
tempvar my_ap = get_ap_res.ap_val;

let fp_and_pc = get_fp_and_pc();
tempvar my_fp = fp_and_pc.fp_val;
tempvar my_pc = fp_and_pc.pc_val;
```

（您将在下面的部分中了解有关此语法的更多信息。）

当 Cairo 需要在复合表达式中使用地址 `fp` 时，它会尝试将其替换为名为 `__fp__` 的变量，假定该变量包含 `fp` 的值。 请注意，对 `fp` 的取消引用（例如 `[fp + 3]`）总是可以的。 例如，以下代码中的 B 需要 A 才能编译，而 C 不需要。

```
local __fp__: felt* = fp_and_pc.fp_val;  // A.
tempvar x = fp;  // B.
tempvar y = [fp];  // C.
```

## 函数参数和返回值

以下是获取两个值 x 和 y 并返回它们的和 z 和乘积 w 的函数示例：

```
func foo(x, y) -> (z: felt, w: felt) {
    [ap] = x + y, ap++;  // z.
    [ap] = x * y, ap++;  // w.
    ret;
}
```

在调用指令之前将参数写入“堆栈”。 例如，要调用 foo(4, 5) 应该写成：

```
[ap] = 4, ap++;  // x.
[ap] = 5, ap++;  // y.
call foo;
```

## 函数参数

`call` 调用将另外两个值压入堆栈（下一个 pc 和当前 fp）。 因此，当一个函数启动时，参数在 `[fp - 3]`、`[fp - 4]`、... 处可用（以相反的顺序）。 对于每个参数，Cairo 编译器创建一个对其值的引用 `argname` 和一个常量 `Args.argname` 及其偏移量 (0, 1, 2, ...)。 引用 `argname` 的任何用法都将替换为 `[fp - (2 + n_args) + Args.argname]`。 这样，您只需编写 `x` 即可访问名为 `x` 的参数的值。

Cairo 支持以下语法糖来调用函数，该函数还支持[复合表达式]()：

```
foo(x=4, y=5);
```

### 返回值

该函数在 ret 指令之前将其返回值写入堆栈。 因此，在函数调用之后，调用者可以在 `[ap - 1]`、`[ap - 2]` 等处获得返回值。

例如，要使用 foo 返回的值，您可以编写：

```
foo(x=4, y=5);
[ap] = [ap - 1] + [ap - 2], ap++;  // 计算 z + w.
```
Cairo 编译器自动创建一个名为 foo.Return 的类型定义，返回类型为：(z: feel, w: feel)。 实际上，可以如下定义类型化引用：let foo_ret = [cast(ap - 2, foo.Return*)];。 现在，您可以访问 z 作为 foo_ret.z。

Cairo 支持这些情况的语法糖（我们称之为“返回值引用”）：

```
let foo_ret = foo(x=4, y=5);
// foo_ret 是对 (ap - 2) 类型的隐式引用
[ap] = foo_ret.z + foo_ret.w, ap++;
```

### 返回值解包

Cairo 支持通过元组将多个返回值分配给引用的语法糖。 语法 let (z, w) = foo(x=4, x=5); 将 foo 的返回值分别赋给 z 和 w：

```
let (z, w) = foo(x=4, y=5);
[ap] = z + w, ap++;
```

在许多情况下，您可能希望将结果复制到局部变量，以防止它在以后被撤销。 虽然您可以添加一条指令 local z = z;，它将引用重新绑定到具有相同名称的新局部变量，但可以使用以下方法实现相同的效果：

```
let (local z, local w) = foo(x=4, y=5);
[ap] = z + w, ap++;
```

## 命名参数

在许多情况下，让编译器警告函数定义和函数调用中的参数列表之间的不一致是有帮助的。 例如，如果添加了一个函数参数，如果在调用函数时未传递该参数，您可能希望得到一个错误。 要允许编译器生成该警报，请在调用函数时使用以下模式：

```
let args = cast(ap, foo.Args*);
args.x = 4, ap++;
args.y = 5, ap++;
// 检查 ap 是否执行了正确的次数（这将确保不会忘记参数）。
static_assert args + foo.Args.SIZE == ap;
let foo_ret = call foo;
```

请注意，这种方式可以按任何顺序传递参数（例如，在 x 之前传递 y）。

### 练习
通过将参数 x 重命名为 new_x 来修改函数 foo（不要修改调用代码）。

对 foo 中添加/删除参数执行相同的操作。

## 尾递归
使用上述方法可以有效地进行尾递归。 尾递归是指函数通过调用第二个函数结束并立即返回此内部函数的输出而不进行任何修改的情况。 例如，以 return sin(2 * x); 结尾的函数。 使用尾递归，但函数以 return 2*sin(x); 才不是。 在这种情况下使用以下模式：

```
call inner_func;
ret;
```

尾调用的高级语法等价物是 return inner_func(...) （请参阅 Return tuple）：

```
return inner_func(x=4, y=5);
```

在这两种情况下，inner_func 的返回值都由调用函数传递。

### 练习
阅读以下斐波那契程序：

```
func main() {
    // 嗲用 fib(1, 1, 10).
    [ap] = 1, ap++;
    [ap] = 1, ap++;
    [ap] = 10, ap++;
    call fib;

    // 第十个斐波那契数是 144.
    [ap - 1] = 144;
    ret;
}

func fib(first_element, second_element, n) -> (res: felt) {
    jmp fib_body if n != 0;
    [ap] = second_element, ap++;
    ret;

    fib_body:
    [ap] = second_element, ap++;
    [ap] = first_element + second_element, ap++;
    [ap] = n - 1, ap++;
    call fib;
    ret;
}
```

确保您了解内存布局、ap 和 fp 寄存器的使用以及尾递归返回值的概念。

### 练习
1.实现函数 $ f(x,n) = x^n $ 使用递归规则 $ f(x,n+1) = f(x,n)*x $

2.添加调用 x=2, n=7 的函数的代码，运行它（如果你得到 End of program was not reached 错误，增加步骤数）并验证结果（例如，通过使用 `--print_memory` 或通过 添加一个假断言指令 `[ap - 1] = 1111` 并确保错误显示类似于 ASSERT_EQ 指令失败：128 ！= 1111）。

3.您的程序的运行时间是多少（即作为 n 函数的确切步数）？ 先猜测或计算，然后通过添加虚假的错误断言并使用 `--debug_error` `-`-print_info` 运行来计算它

## 返回元组

Cairo 支持以下语法糖，可以轻松地从函数返回值：

```
func foo() -> (a: felt, b: felt) {
    return (a=<expr0>, b=<expr1>);
}
```

等价于

```
func foo() -> (a: felt, b: felt) {
    [ap] = <expr0>, ap++;
    [ap] = <expr1>, ap++;
    ret;
}
```

根据声明的返回类型检查命名参数。 请注意，返回值支持[复合表达式]()。