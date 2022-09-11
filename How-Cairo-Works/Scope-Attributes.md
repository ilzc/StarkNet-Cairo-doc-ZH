
# 作用域属性
您可以通过用 with_attr 语句包围代码块来定义代码块的属性，如下所示：

```
with_attr attribute_name("Attribute value") {
    // 代码块
}
```

属性值必须是字符串，并且只能引用局部变量。 通过将变量名放在大括号内来引用变量（例如，“x 必须是正数。得到：{x}。”）。

目前 Cairo 只支持一个属性：`error_message`。 它允许用户使用信息丰富的错误消息来注释代码块。 如果运行时错误源于此属性包装的代码，VM 将自动将相应的错误消息添加到错误跟踪中。

请注意，如果导致错误的语句被多个 `with_attr` 包围，则所有错误消息都将出现在输出中。 对于包装函数调用的错误消息也是如此。 考虑以下示例：

```
from starkware.cairo.common.math import assert_not_zero

func inverse(x) -> (res: felt) {
    with_attr error_message("x必须不为0. x={x}.") {
        return (res=1 / x);
    }
}

func assert_not_equal(a, b) {
    let diff = a - b;
    with_attr error_message("a和b不相等.") {
        inverse(diff);
    }
    return ();
}
```

如果你调用 `assert_not_equal` 传入参数 `a == b`，你应该得到以下错误：

```
Error message: x must not be zero. Got x=0.
:5:21: Error at pc=0:7:
Unknown value for memory cell at address 1:8.
        return (res=1 / x);
                    ^***^
Cairo traceback (most recent call last):
Error message: a and b must be distinct.
:12:9: (pc=0:10)
        inverse(diff);
        ^***********^
```