# 调用其他合约

一个合约函数可以调用另一个合约的外部函数。

首先部署编写 ```StarkNet``` 合约中的示例合约（编译和部署说明可以在页面底部找到）。 用 ```BALANCE_CONTRACT``` 表示这个合约的地址。

为了从另一个合约调用这个合约，通过复制外部函数的声明来定义一个接口：

```
@contract_interface
namespace IBalanceContract {
    func increase_balance(amount: felt) {
    }

    func get_balance() -> (res: felt) {
    }
}
```
请注意，应从定义中删除函数体和隐式参数。

您可以使用 ```IBalanceContract.increase_balance()``` 和 ```IBalanceContract.get_balance()``` 在另一个合约上调用这些函数。 例如：

```
@external
func call_increase_balance{syscall_ptr: felt*, range_check_ptr}(
    contract_address: felt, amount: felt
) {
    IBalanceContract.increase_balance(
        contract_address=contract_address, amount=amount
    );
    return ();
}

@view
func call_get_balance{syscall_ptr: felt*, range_check_ptr}(
    contract_address: felt
) -> (res: felt) {
    let (res) = IBalanceContract.get_balance(
        contract_address=contract_address
    );
    return (res=res);
}
```

请注意，调用另一个合约的函数需要在函数的原始参数之前传递一个额外的参数——被调用合约的地址。 例如，```IBalanceContract.increase_balance``` 有两个参数：```contract_address``` 和 ```amount```（而不仅仅是金额）。 此外，还需要 ```syscall_ptr``` 和 ```range_check_ptr``` 隐式参数。

创建一个名为 ```proxy_contract.cairo``` 的文件，其中包含接口声明和两个函数 ```call_increase_balance()``` 和 ```call_get_balance()```，并部署合约。 用 ```PROXY_CONTRACT``` 表示新合约的地址。

现在，使用 ```BALANCE_CONTRACT``` 作为 ```contract_address``` 参数的值调用 ```call_increase_balance。``` 确保将 ```PROXY_CONTRACT``` 和 ```BALANCE_CONTRACT``` 替换为部署两个合约时获得的地址：

```
starknet invoke \
    --address PROXY_CONTRACT \
    --abi proxy_contract_abi.json \
    --function call_increase_balance \
    --inputs BALANCE_CONTRACT 10000
```

这将增加存储在 ```BALANCE_CONTRACT``` 中的余额。 请注意，在我们的例子中，```PROXY_CONTRACT``` 的存储不会受到影响。

等到交易被添加到区块中，然后使用以下两种方式查看余额：

1. 直接通过 ```BALANCE_CONTRACT```
```
starknet call \
    --address BALANCE_CONTRACT \
    --abi balance_contract_abi.json \
    --function get_balance
```

2. 间接通过 ```PROXY_CONTRACT```
```
starknet call \
    --address PROXY_CONTRACT \
    --abi proxy_contract_abi.json \
    --function call_get_balance \
    --inputs BALANCE_CONTRACT
```

两个命令都应该返回 10000

## 获取当前合约的地址

您可以使用 get_contract_address() 库函数获取当前合约的地址。

```
from starkware.starknet.common.syscalls import (
    get_contract_address,
)

// ...

let (contract_address) = get_contract_address();
```

以上与 Solidity 中的 address(this) 类似。

## 库调用

库调用是一种在调用合约的上下文中调用在给定合约类中声明的函数的方法。 可以调用在任何先前声明的类中定义的函数（特别是已经部署的任何合约的类）。

调用函数中的存储更改操作将更改调用合约的状态。 类似地，```get_caller_address()``` 和 ```get_contract_address()``` 将返回与调用函数调用时相同的值。

请注意，库调用与以太坊的委托调用非常相似。 不同之处在于委托调用需要已部署的合约，而库调用与合约类一起使用（无论如何，以太坊的委托调用中不使用合约实例）。

要执行库调用，请使用上述合约接口，但在函数名称前添加 ```library_call_``` 并传递类哈希而不是合约地址。

```
// 在我们的代理合约中定义一个本地余额变量。
@storage_var
func balance() -> (res: felt) {
}

@external
func increase_my_balance{syscall_ptr: felt*, range_check_ptr}(
    class_hash: felt, amount: felt
) {
    // 通过使用库调用，使用来自不同合约类的函数增加本地余额变量。
    IBalanceContract.library_call_increase_balance(
        class_hash=class_hash, amount=amount
    );
    return ();
}
```

通过添加上面的代码来修改您之前创建的文件 ```proxy_contract.cairo```。 重新编译并重新部署这个新版本的代理合约，并用 ```PROXY_CONTRACT``` 表示它的地址。

接下来，声明一个库类，在我们的例子中——已经编译的示例余额合约。

```
starknet declare --contract balance_contract_compiled.json
```

用 ```BALANCE_CLASS_HASH``` 表示这个类的散列。 现在，调用 ```increase_my_balance```：

```
starknet invoke \
    --address PROXY_CONTRACT \
    --abi proxy_contract_abi.json \
    --function increase_my_balance \
    --inputs BALANCE_CLASS_HASH 12321
```

这会使用 balance 合约类的 ```increase_balance()``` 函数增加代理合约存储中的余额。 与常规合约调用不同，此处调用合约（而不是另一个合约）的余额被修改。

注意：您可以使用库调用来调用一个函数，该函数会更改调用合约中未定义的存储变量。 在这种情况下，新的相应存储变量将在调用合约中创建，但不容易访问（您可以通过第二个库调用访问它，或直接使用 ```storage_read()```）。