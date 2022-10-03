# 更多特性

## 存储变量保存多个值

存储变量不必是单个字段元素，也可以是多个字段元素的元组。 例如：

```
// 一个从键 user 到值 (min, max) 的映射.
@storage_var
func range(user: felt) -> (res: (felt, felt)) {
}
```

你可以这样对存储变量进行读写

```
@external
func extend_range{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(user: felt) {
    let (min_max) = range.read(user);
    range.write(user, (min_max[0] - 1, min_max[1] + 1));
    return ();
}
```

请注意，在这种情况下 range.read() 返回一个是 pair 值。 因此，让 (min, max) = range.read(user); 不管用。

## 存储变量保存 struct 参数

存储变量的参数也可以是结构体或元组，只要它们不包含指针（这种不包含指针的类型称为 felt 类型）。 例如：

```
struct User {
    first_name: felt,
    last_name: felt,
}

// 一个映射从 user 到 整形
@storage_var
func user_voted(user: User) -> (res: felt) {
}

@external
func vote{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(user: User) {
    user_voted.write(user, 1);
    return ();
}
```

## calldata 中的数组参数

外部函数可以获取字段元素数组作为参数。 为了定义一个名为 ```a``` 的数组，传递两个连续的参数：felt 类型的 a_len 和 felt* 类型的 a（如果第二个参数命名为 a，则第一个参数必须命名为 a_len）。 例如：

```
@external
func compare_arrays(
    a_len: felt, a: felt*, b_len: felt, b: felt*
) {
    assert a_len = b_len;
    if (a_len == 0) {
        return ();
    }
    assert a[0] = b[0];
    return compare_arrays(
        a_len=a_len - 1, a=&a[1], b_len=b_len - 1, b=&b[1]
    );
}
```

为了使用数组 [10, 20, 30, 40] 和 [50, 60] 调用 ```compare_arrays```，您应该将以下参数输入传递给 ```starknet invoke``` 

```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi contract_abi.json \
    --function compare_arrays \
    --inputs 4 10 20 30 40 2 50 60
```

第一个值 4 是第一个数组的长度，然后是它的 4 个整数。 之后，我们得到第二个数组 (2) 的长度，然后是它的条目。 请注意，使用上述参数调用 compare_arrays 将失败，因为数组不同。

在外部函数中使用数组参数的 StarkNet 合约必须具有 range_check 内置函数，用于验证数组的长度是否为非负数。

## calldata 中传递元组和结构体

calldata 参数和返回值可以是任何不包含指针的类型。 例如，具有 felt 的结构体、 felt 元组和包含 felt 元组的元组。 例如：

```
struct Point {
    x: felt,
    y: felt,
}

@view
func sum_points(points: (Point, Point)) -> (res: Point) {
    return (
        res=Point(
        x=points[0].x + points[1].x,
        y=points[0].y + points[1].y),
    );
}
```

为了使用点 (1, 2), (10, 20) 调用 ```sum_points```，您应该将以下输入传递给 ```starknet invoke```：

```
starknet call \
    --address ${CONTRACT_ADDRESS} \
    --abi contract_abi.json \
    --function sum_points \
    --inputs 1 2 10 20
```

## 传入结构体数组

类似的方式，只要结构不包含指针，就支持传递结构数组：

```
@external
func sum_points_arr(a_len: felt, a: Point*) -> (res: Point) {
    if (a_len == 0) {
        return (res=Point(0, 0));
    }
    let (res) = sum_points_arr(a_len=a_len - 1, a=&a[1]);
    return (res=Point(x=res.x + a[0].x, y=res.y + a[0].y));
}
```

为了使用 3 个点 (1, 2), (10, 20), (100, 200) 调用 ```sum_points_arr```，您应该将以下输入传递给 ```starknet invoke```：

```
starknet call \
    --address ${CONTRACT_ADDRESS} \
    --abi contract_abi.json \
    --function sum_points_arr \
    --inputs 3 1 2 10 20 100 200
```

## 获取交易信息

您可以使用 ```get_tx_info()``` 库函数检索交易信息（例如，包括签名和交易费用）：

```
from starkware.starknet.common.syscalls import get_tx_info

func get_tx_max_fee{syscall_ptr: felt*}() -> (max_fee: felt) {
    let (tx_info) = get_tx_info();

    return (max_fee=tx_info.max_fee);
}
```

返回值是一个指向 TxInfo 结构的指针，定义如下：

```
struct TxInfo {
    // 交易版本
    version: felt,

    // 发起此交易的账户合约.
    account_contract_address: felt,

    // 交易的 max_fee 字段。
    max_fee: felt,

    // 交易的签名字段。
    signature_len: felt,
    signature: felt*,

    // 交易的哈希值字段。
    transaction_hash: felt,

    // 交易的chain_id字段，却分不同的网络。
    chain_id: felt,

    // 交易的nonce。
    nonce: felt,
}
```

## 区块号和时间戳

您可以使用 ```get_block_number()``` 和 ```get_block_timestamp()``` 库函数获取当前块号和时间戳（自 unix 纪元以来的秒数）。

```
from starkware.starknet.common.syscalls import (
    get_block_number,
    get_block_timestamp,
)

// ...

let (block_number) = get_block_number();
let (block_timestamp) = get_block_timestamp();
```

请注意，上述两个函数都需要隐式参数 ```syscall_ptr```。 目前，```get_block_timestamp()``` 的结果并未由 ```StarkNet``` 操作系统或核心合约强制执行（即，定序器可以选择任意时间戳）。 未来会增加对新时间戳的一些限制。 另请注意，块时间戳是块创建开始的时间，与 L1 上接受块的时间可能有很大不同。