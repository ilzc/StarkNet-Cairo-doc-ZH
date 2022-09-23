# 添加用户认证

## 存储映射

假设我们不想维护一个全局变量 ```balance```，而是希望每个用户都有一个余额（用户将通过他们的 STARK 公钥来识别）。

我们的第一个任务是将余额存储变量更改为从公钥（用户）到余额（而不是单个值）的映射。 这可以通过简单地添加一个参数来完成：

```
// 一个用户映射余额的例子(以账户合约地址表示)
@storage_var
func balance(user: felt) -> (res: felt) {
}
```

事实上，```@storage_var``` 装饰器允许您添加多个参数来创建更复杂的映射。 函数 ```balance.read()``` 和 ```balance.write()``` 现在将具有以下签名：

```
func read{
    syscall_ptr: felt*,
    range_check_ptr,
    pedersen_ptr: HashBuiltin*,
}(user: felt) -> (res: felt) {
}

func write{
    syscall_ptr: felt*,
    range_check_ptr,
    pedersen_ptr: HashBuiltin*,
}(user: felt, value: felt) {
}
```
请注意，映射中所有条目的默认值为 0。

## 获取调用者地址

为了获取调用我们函数的账户合约（或任何其他合约，如果该函数被合约调用）的地址，我们可以使用 ```get_caller_address()``` 库函数：

```
from starkware.starknet.common.syscalls import get_caller_address

// ...

let (caller_address) = get_caller_address();
```

```get_caller_address()``` 返回调用该合约的源合约的地址。 它可以是账户合约的地址，也可以是另一个合约的地址（如果该函数被另一个合约调用）。 当直接调用合约（而不是通过合约）时，函数返回 0。

请注意，如果您在由合约中的另一个函数 ```bar()``` 调用的函数 ```foo()``` 中使用 ```get_caller_address()```，它仍将返回调用 ```bar()``` 的合约的地址（如果直接调用，则返回 0） .

## 修改合约功能

将 ```increase_balance()``` 的代码更改为：
```
from starkware.cairo.common.math import assert_nn

// 将用户的余额增加特定的数量。
@external
func increase_balance{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(amount: felt) {
    // 验证金额是否为正。
    with_attr error_message(
            "Amount must be positive. Got: {amount}.") {
        assert_nn(amount);
    }

    // 获取账户合约地址。
    let (user) = get_caller_address();

    // 读取并更新余额
    let (res) = balance.read(user=user);
    balance.write(user, res + amount);
    return ();
}
```

请注意，我们通过调用 assert_nn 添加了一个约束，即数量的值必须为非负数。 为了在发生错误时获得指示性消息，我们使用 ```with_attr error_message(...)``` 区块包装了函数调用。 有关更多详细信息，请参阅[此处]()。

同样，更改 ```get_balance()``` 的代码。 在这里，我们选择允许调用者查询任何用户（因为 StarkNet 的存储无论如何都不是私有的）：

```
// 返回指定用户的余额
@view
func get_balance{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(user: felt) -> (res: felt) {
    let (res) = balance.read(user=user);
    return (res=res);
}
```

## 编译和部署
将新合约文件另存为 ```user_auth.cairo```。 您可以在[此处](https://starknet.io/docs/_static/user_auth.cairo)找到完整的Cairo 文件。

编译和部署文件：

```
starknet-compile user_auth.cairo \
    --output user_auth_compiled.json \
    --abi user_auth_abi.json

starknet deploy --contract user_auth_compiled.json --no_wallet
```

在运行 ```starknet deploy``` 命令之前，不要忘记设置 ```STARKNET_NETWORK``` 和 ```STARKNET_WALLET``` 环境变量并部署帐户合约。

## 和合约交互
让我们更新合约余额
```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi user_auth_abi.json \
    --function increase_balance \
    --inputs 4321
```
您可以查询交易状态：
```
starknet tx_status --hash TX_HASH
```
最后，在交易执行后（状态 ACCEPTED_ON_L2 或 ACCEPTED_ON_L1）您可以查询用户的余额：
```
starknet call \
    --address ${CONTRACT_ADDRESS} \
    --abi user_auth_abi.json \
    --function get_balance \
    --inputs ${ACCOUNT_ADDRESS}
```
你应该得到：
```
4321
```
请注意，如果您想使用 ``get_storage_at`` CLI 命令查询特定用户的余额，您不能再仅通过提供存储变量的名称来计算相关键。 这是因为余额存储变量现在需要一个额外的参数，即用户密钥。 因此，在获取 ```get_storage_at``` 中使用的密钥时，您需要提供额外的参数。 在我们的例子中，这转换为以下 Python 代码：
```
from starkware.starknet.public.abi import get_storage_var_address

user = ACCOUNT_ADDRESS
user_balance_key = get_storage_var_address('balance', user)
print(f'Storage key for user {user}:\n{user_balance_key}')
```

## 获取回退的原因
让我们尝试使用负数调用 ```increase_balance：```
```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi user_auth_abi.json \
    --function increase_balance \
    --inputs -1000
```
由于此交易无效（因为金额为负数），您将收到来自 StarkNet 网关的错误，其中包含以下内容：
```
{"code": "StarknetErrorCode.TRANSACTION_FAILED", "message": "Error at pc=0:38:\nGot an exception while executing a hint.\nCairo traceback (most recent call last):\nUnknown location (pc=0:522)\nUnknown location (pc=0:484)\nUnknown location (pc=0:590)\n\nError in the called contract (0x548245153813267af2d2793c6e5d60c40cb95f34d7404f2ce75550fafabede0):\nError at pc=0:6:\nGot an exception while executing a hint.\nCairo traceback (most recent call last):\nUnknown location (pc=0:155)\nError message: Amount must be positive. Got: -1000.\nUnknown location (pc=0:129)\n\nTraceback (most recent call last):\n  File \"<hint0>\", line 3, in <module>\nAssertionError: a = 3618502788666131213697322783095070105623107215331596699973092056135872019481 is out of range."}
```

这表明 CLI 无法估算交易费用，因为交易失败。 为了演示检索revert原因，我们将强制交易跳过费用估算机制。 为此，将 ```--max_fee 100000000000000000``` 添加到前一个调用事务中，如下所示：
```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi user_auth_abi.json \
    --function increase_balance \
    --inputs -1000 \
    --max_fee 100000000000000000
```
在此之后，查询交易状态时，您应该得到：
```
{
    "tx_failure_reason": {
        "code": "TRANSACTION_FAILED",
        "error_message": "Error at pc=0:32:\nGot an exception while executing a hint.\nCairo traceback (most recent call last):\nUnknown location (pc=0:494)\nUnknown location (pc=0:453)\nUnknown location (pc=0:510)\n\nError in the called contract (0x3632c8d1265888e0eadb518cbf4a83d071d00cd8f946ec72fd661e69eea1963):\nError at pc=0:6:\nGot an exception while executing a hint.\nCairo traceback (most recent call last):\nUnknown location (pc=0:155)\nError message: Amount must be positive. Got: -1000.\nUnknown location (pc=0:129)\n\nTraceback (most recent call last):\n  File \"<hint0>\", line 3, in <module>\nAssertionError: a = 3618502788666131213697322783095070105623107215331596699973092056135872019481 is out of range."
    },
    "tx_status": "REJECTED"
}
```

请注意，错误消息条目指出错误位置未知。 这是因为 StarkNet 网络不知道合约的源代码和调试信息。 要检索错误位置并重建回溯，请使用 ```--contracts``` 参数在交易状态查询中添加相关已编译合约的路径。 为了更好地显示错误（并且只显示错误），还要添加 ```--error_message``` 标志：

```
starknet tx_status \
    --hash TX_HASH \
    --contracts ${CONTRACT_ADDRESS}:user_auth_compiled.json \
    --error_message
```

输出应如下所示：
```
Error at pc=0:28:
Got an exception while executing a hint.
Cairo traceback (most recent call last):
Unknown location (pc=0:494)
Unknown location (pc=0:453)
Unknown location (pc=0:510)

Error in the called contract (0x29cd5db92729052b3268471cf1b2327b61523565adeaa1d659236e806bd4b97):
math.cairo:47:5: Error at pc=0:6:
    a = [range_check_ptr];
    ^*******************^
Got an exception while executing a hint.
Cairo traceback (most recent call last):
user_auth.cairo:15:6
func increase_balance{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
     ^**************^
Error message: Amount must be positive. Got: -1000.
user_auth.cairo:20:9
        assert_nn(amount);
        ^***************^

Traceback (most recent call last):
  File "<hint0>", line 3, in <module>
AssertionError: a = 3618502788666131213697322783095070105623107215331596699973092056135872019481 is out of range.
```

你应该忽略第一部分（在 ```Error in the called contract``` 之前）——它是由账户合约引起的。