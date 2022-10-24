# 事件

StarkNet 事件机制允许合约在其执行期间输出信息，以后可以在 StarkNet 之外使用。 例如，考虑一个允许在用户之间转移代币的代币合约。 通过仅查询 StarkNet 存储，用户只能看到他们拥有多少代币，而不是谁转移了这些代币。 每当资金转移时由合约发出的事件可用于允许用户获取此信息。

考虑添加用户身份验证中描述的合同。 每当调用 increase_balance() 函数时，让我们添加一个事件。

首先定义事件：
```
// 当调用 increase_balance() 触发事件.
// current_balance 增加余额前的值.
@event
func increase_balance_called(
    current_balance: felt, amount: felt
) {
}
```

在 ```increase_balance()``` 中的 ```return()``` 语句之前添加以下行：

```
//触发事件
increase_balance_called.emit(current_balance=res, amount=amount);
```

将新的合约文件保存为 events.cairo。 你可以在这里找到完整的[合约](https://starknet.io/docs/_static/events.cairo)。

编译并声明合约：
```
starknet-compile events.cairo \
    --output events_compiled.json \
    --abi events_abi.json
starknet declare --contract events_compiled.json
```

部署合约：
```
starknet deploy --class_hash ${EVENTS_CLASS_HASH}
```

其中 ```${EVENTS_CLASS_HASH}``` 是 ```class_hash``` 的值。 调用 ```increase_balance()```：

```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi events_abi.json \
    --function increase_balance \
    --inputs 4321
```

等待交易被接受（至少当其状态为 ```PENDING``` 时）并运行以下行以查看发出的事件（将 ```${TRANSACTION_HASH}``` 替换为您从最后一个命令获得的交易哈希）：

```
starknet get_transaction_receipt --hash ${TRANSACTION_HASH}
```

考虑输出的事件部分。 它应该类似于：

```
"events": [
    {
        "data": [
            "0x0",
            "0x10e1"
        ],
        "from_address": "0x14acf3b7e92f97adee4d5359a7de3d673582f0ce03d33879cdbdbf03ec7fa5d",
        "keys": [
            "0x3db3da4221c078e78bd987e54e1cc24570d89a7002cefa33e548d6c72c73f9d"
        ]
    }
]
```

结果包含以下字段：

* from_address – 发出事件的合约的地址。

* data – 传递给 increase_balance_call.emit 的参数：(0) 之前的余额和金额 (4321==0x10e1)。

* key – 事件的密钥，它源自事件的名称 (increase_balance_call)。 如果你的合约发出不止一种类型的事件，你可以使用这个字段来区分它们。 您可以使用 Python 从其名称中获取事件键：

```
from starkware.starknet.compiler.compile import \
    get_selector_from_name

print(hex(get_selector_from_name('increase_balance_called')))
```

请注意，StarkNet 目前没有从给定合约中获取所有事件的 API。