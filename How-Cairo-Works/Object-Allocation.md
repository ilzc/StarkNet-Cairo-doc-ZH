# 对象分配
Cairo 有几种方法可以将对象存储在内存中并获取指向它的指针（其他函数可以调用）：

1.内存段分配：`alloc()` 函数可用于创建任意长度的数组。

2.单个元素分配：new 运算符初始化单个元素并返回一个指向它的指针。

3.局部变量：您可以分配局部变量并检索其地址。请参阅访问[寄存器]()的值。

## [`alloc()`](#alloc)
标准库函数 alloc() 可用于“动态”分配新的内存段（参见 [段]()）。该段可用于存储数组或单个元素。

例如:
从 starkware.cairo.common.alloc 导入 `alloc`

```
func foo() {
    let (struct_array: MyStruct*) = alloc();

    // 设置前三个元素。
    assert struct_array[0] = MyStruct(a=1, b=2);
    assert struct_array[1] = MyStruct(a=3, b=4);
    assert struct_array[2] = MyStruct(a=5, b=6);
    return ();
}
```

## [`new` 操作符](#new)
new 运算符接受一个表达式并将其压入堆栈，并返回一个指向该对象的内存地址的指针。例如：

```
func foo() {
    tempvar ptr: MyStruct* = new MyStruct(a=1, b=2);
    assert ptr.a = 1;
    assert ptr.b = 2;
    return ();
}
```

请注意，与分配新内存段的 `alloc()` 不同，`new` 运算符在执行段中创建对象（请参阅 [段]()）。由于 Cairo 中的内存永远不会被释放，因此两种方法都有相似的结果——即使在函数结束后你也可以使用指针。另一方面，这意味着 new 运算符不能用于任意大小的数组。 `new` 运算符很有用，因为它允许您在一条指令中分配内存并初始化对象。事实上，如有必要，您可以在同一行中使用多个新运算符。

这在理论上等价于以下代码：

```
tempvar obj: MyStruct = MyStruct(a=1, b=2);
tempvar ptr: MyStruct* = &obj;
```

但是，tempvar 当前不支持 &obj，因此无法编译此代码。

您可以使用 `new` 使用元组分配固定大小的数组：

```
func foo() {
    tempvar arr: felt* = new (1, 1, 2, 3, 5);
    assert arr[4] = 5;
    return ();
}
```

对于结构数组，您需要显式转换指针：

```
func foo() {
    tempvar arr: MyStruct* = cast(
        new (MyStruct(a=1, b=2), MyStruct(a=3, b=4)), MyStruct*);
    assert arr[1].a = 3;
    return ();
}
```
