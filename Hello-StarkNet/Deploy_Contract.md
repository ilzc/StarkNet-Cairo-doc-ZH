# 通过另一个合约部署一个合约

## 系统调用 deploy

一个合约可以使用 ```deploy``` 系统调用来部署另一个合约。 注意，部署合约的合约类必须在调用系统调用的事务之前声明。 ```deploy``` 系统调用定义如下：

```
func deploy{syscall_ptr: felt*}(
    class_hash: felt,
    contract_address_salt: felt,
    constructor_calldata_size: felt,
    constructor_calldata: felt*,
    deploy_from_zero: felt,
) -> (contract_address: felt) {
}
```

如上所示，部署系统调用需要以下参数：

* class_hash：要部署的合约的类哈希。

* contract_address_salt：用于确定新合约地址的任意值。 使用相同的 salt、相同的 class_hash 和来自相同合约的相同构造函数参数将导致相同的地址，这意味着第二次部署将失败。

* constructor_calldata_size：构造函数参数（calldata）的大小。 请注意，在并非所有参数都是 felt 的情况下，这可能与参数的数量不同。

* constructor_calldata：指向包含构造函数参数的数组的指针。

* deploy_from_zero：决定部署者地址是否会影响合约地址的标志。 当等于 TRUE 时，合约地址计算将使用 0 代替部署者地址。 当等于 FALSE 时，将使用实际部署者的地址。

系统调用返回contract_address：新部署的合约的地址。

考虑构造函数中的可拥有合约。 以下合约是使用 deploy 系统调用来部署可拥有合约的新实例的示例：

```
%lang starknet

from starkware.cairo.common.alloc import alloc
from starkware.cairo.common.bool import FALSE
from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.syscalls import deploy

// 提供一个存储变量记录了 salt .
@storage_var
func salt() -> (value: felt) {
}

// 定义一个存储变量存储class hash ownable_contract.
@storage_var
func ownable_class_hash() -> (value: felt) {
}

// An event emitted whenever deploy_ownable_contract() is called.
@event
func ownable_contract_deployed(contract_address: felt) {
}

@constructor
func constructor{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(ownable_class_hash_: felt) {
    ownable_class_hash.write(value=ownable_class_hash_);
    return ();
}

@external
func deploy_ownable_contract{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(owner_address: felt) {
    let (current_salt) = salt.read();
    let (class_hash) = ownable_class_hash.read();
    let (contract_address) = deploy(
        class_hash=class_hash,
        contract_address_salt=current_salt,
        constructor_calldata_size=1,
        constructor_calldata=cast(new (owner_address,), felt*),
        deploy_from_zero=FALSE,
    );
    salt.write(value=current_salt + 1);

    ownable_contract_deployed.emit(
        contract_address=contract_address
    );
    return ();
}
```

请注意，仅当先前声明了所需的类时，上述内容才有效。

## 使用合约

将新合约文件保存为 ```ownable_contract_deployer.cairo```

按照构造函数中的说明编译和声明可拥有合约。 请注意，您不需要部署合约的实例，在使用 ```deploy``` 系统调用时声明合约类就足够了。 在 ```StarkNet``` 测试网上声明合约中查看有关声明交易的更多详细信息。

将名为 ```OWNABLE_CLASS_HASH``` 的环境变量设置为 ```ownable_contract.cairo``` 的类哈希。

编译并声明 ```ownable_contract_deployer.cairo```：

```
starknet-compile ownable_contract_deployer.cairo \
    --output ownable_contract_deployer_compiled.json \
    --abi ownable_contract_deployer_abi.json
starknet declare --contract ownable_contract_deployer_compiled.json
```

部署合约:
```
starknet deploy --class_hash ${OWNABLE_DEPLOYER_CLASS_HASH} \
    --inputs ${OWNABLE_CLASS_HASH}
```

其中 ```${OWNABLE_DEPLOYER_CLASS_HASH}``` 是 ```class_hash``` 的值。 为所有者地址选择任意值，并用 ```OWNER_ADDRESS``` 表示。 现在，使用 ```OWNER_ADDRESS``` 作为 ```owner_address``` 参数的值调用 ```deploy_ownable_contract```。 确保 ```CONTRACT_ADDRESS``` 环境变量设置为您在部署合约时获得的地址：

```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi ownable_contract_deployer_abi.json \
    --function deploy_ownable_contract \
    --inputs OWNER_ADDRESS
```

这将部署一个以 ```OWNER_ADDRESS``` 作为所有者地址的新 ```ownable_contract。```

合约的地址被发出，并且可以通过交易收据看到，如事件中所述。 事件部分中的数据包含已部署合约的地址。 收据的事件部分应如下所示：

```
"events": [
    {
        "data": [
            "0x338027db29a197a7d5dbd49f1e15c9b6702d6a16758dda905efc751bb117153"
        ],
        "from_address": "0x7569242709918b8929078d3178ed14588348fb5459b44a41f100eb9a67dbeb6",
        "keys": [
            "0x2902eb93dff1da1a2de652946319fafe27b03601628834219f8738fc9b361d7"
        ]
    }
]
```

使用以下命令查询所有者地址。 将 ```OWNABLE_CONTRACT_ADDRESS``` 替换为部署的 ```ownable_contract``` 的地址：

```
starknet call \
    --address OWNABLE_CONTRACT_ADDRESS \
    --abi ownable_abi.json \
    --function get_owner
```

返回的值应该是之前使用的 ```OWNER_ADDRESS```

重要提示：将来，使用 ```deploy``` 系统调用将是部署新合约的唯一方法。