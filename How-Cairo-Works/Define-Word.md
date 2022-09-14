# 字面常量

`dw` 关键字直接在代码中编译为数据的单个字段元素。 例如：

```
dw 0x123;
```

将被翻译为字节码中的字段元素 0x123。 这不是真正的指令，因此不应在任何执行路径中。 一个常见的用途是常量数组：

```
from starkware.cairo.common.registers import get_label_location

// Returns a pointer to the values: [1, 22, 333, 4444].
func get_data() -> (data: felt*) {
    let (data_address) = get_label_location(data_start);
    return (data=cast(data_address, felt*));

    data_start:
    dw 1;
    dw 22;
    dw 333;
    dw 4444;
}

func main() {
    let (data) = get_data();
    tempvar value = data[2];
    assert value = 333;
    return ();
}
```