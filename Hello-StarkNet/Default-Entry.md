# 默认入口

在某些情况下，合约入口是事先不知道的。 最突出的示例是将调用转发到实现合同类的委托代理。 这样的代理可以使用 __default__ 入口点来实现，如下所示：

```
%lang starknet
%builtins pedersen range_check bitwise

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.syscalls import library_call

// 实现class hash
@storage_var
func implementation_hash() -> (class_hash: felt) {
}

@constructor
func constructor{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(implementation_hash_: felt) {
    implementation_hash.write(value=implementation_hash_);
    return ();
}

@external
@raw_input
@raw_output
func __default__{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(selector: felt, calldata_size: felt, calldata: felt*) -> (
    retdata_size: felt, retdata: felt*
) {
    let (class_hash) = implementation_hash.read();

    let (retdata_size: felt, retdata: felt*) = library_call(
        class_hash=class_hash,
        function_selector=selector,
        calldata_size=calldata_size,
        calldata=calldata,
    );
    return (retdata_size=retdata_size, retdata=retdata);
}
```


如果请求的选择器与合约中的任何入口点选择器都不匹配，则执行 ```__default__``` 入口点。

```@raw_input``` 装饰器指示编译器将调用数据按原样传递给入口点，而不是将其解析为请求的参数。在这种情况下，函数的参数必须是 ```selector```、```calldata_size``` 和 ```calldata```。类似地，```@raw_output``` 装饰器指示编译器不要处理函数的返回值。在这种情况下，函数的返回值必须是 ```retdata_size``` 和 ```retdata```

让我们看看如何使用这个代理模式。

创建一个名为 ```balance_contract.cairo``` 的文件，其中包含编写 StarkNet 合约中的示例余额合约代码，并按照在 StarkNet 测试网上声明合约中的说明声明该合约。用 ```${BALANCE_CLASS_HASH}``` 表示新合约类的哈希。

现在，创建一个包含上述代理代码的名为 delegate_proxy.cairo 的文件，并以与 ```balance_contract.cairo``` 相同的方式声明合约。用 ```${DELEGATE_PROXY_CLASS_HASH}``` 表示新合约的哈希值，并部署该合约，其 ```implementation_hash_``` 构造函数参数的值为 ```${BALANCE_CLASS_HASH}```：

```
starknet deploy \
    --class_hash ${DELEGATE_PROXY_CLASS_HASH} \
    --inputs ${BALANCE_CLASS_HASH}
```

用 ```PROXY_CONTRACT``` 表示新合约的地址。

通过委托代理合约调用 ```balance``` 类的 ```increase_balance``` 函数。 您应该将委托代理合约的地址与实现类的 ABI 一起使用，这是定义调用函数的位置。 其余参数与预期一样——函数的输入。

```
starknet invoke \
    --address PROXY_CONTRACT \
    --abi balance_contract_abi.json \
    --function increase_balance \
    --inputs 10000
```

这将增加存储在代理合约中的余额。 请注意，在我们的例子中，实现余额合约只是被声明并没有被部署，所以它没有自己的存储。

与 ```__default__``` 类似，当调用 L1 处理程序但缺少请求的选择器时，将执行 ```__l1_default__``` 入口点。 此入口点与 ```library_call_l1_handler``` 系统调用结合使用可用于转发 L1 处理程序，如下所示：

```
from starkware.starknet.common.syscalls import (
    library_call_l1_handler,
)

@l1_handler
@raw_input
func __l1_default__{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(selector: felt, calldata_size: felt, calldata: felt*) {
    let (class_hash) = implementation_hash.read();

    library_call_l1_handler(
        class_hash=class_hash,
        function_selector=selector,
        calldata_size=calldata_size,
        calldata=calldata,
    );
    return ();
}
```

```library_call_l1_handler``` 系统调用类似于 ```library_call```，只是它调用 l1_handler 入口点而不是外部入口点。 系统调用不像典型用例那样使用 L1 -> L2 消息，相关消息由发出系统调用的 L1 处理程序使用。

请注意，在 L1 处理程序之外调用 ```library_call_l1_handler``` 可能是危险的，因为被调用的处理程序可能会假定使用了适当的消息。