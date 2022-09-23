# CLI 命令

## get_transaction
要获取交易信息，请运行以下命令：
```
starknet get_transaction --hash TRANSACTION_HASH
```

输出应类似于：
```
{
    "block_hash": "0x0",
    "block_number": 0,
    "status": "ACCEPTED_ON_L2",
    "transaction": {
        "calldata": [
            "0x1",
            "0x39564c4f6d9f45a963a6dc8cf32737f0d51a08e446304626173fd838bd70e1c",
            "0x362398bec32bc0ebb411203221a35a0301193a96f317ebe5e40be9f60d15320",
            "0x0",
            "0x1",
            "0x1",
            "0x4d2"
        ],
        "contract_address": "0x78d796e87cfa496bffad27be9ed42f2709bd6e32a6366f842fdf38664a1412d",
        "entry_point_selector": "0x15d40a3d6ca2ac30f4031e42be28da9b056fef9bb7357ac5e85627ee876e5ad",
        "entry_point_type": "EXTERNAL",
        "max_fee": "0x0",
        "nonce": "0x2",
        "signature": [
            "0x26ee1f973def2e6d5c7f32aaad96c84dab32df6a62ee0e8b530a72bc5478fe6",
            "0x502b15e440cc2a8a1966f0c973015096715e366fde36a4659c56f84249688e"
        ],
        "transaction_hash": "0x69d743891f69d758928e163eff1e3d7256752f549f134974d4aa8d26d5d7da8",
        "type": "INVOKE_FUNCTION",
        "version": "0x1"
    },
    "transaction_index": 1
}
```

返回的结果包含：

* transaction_hash – 交易的哈希值。

* status – 交易的状态。 有关支持的事务状态的详细列表，请参阅 tx_status 使用示例。

* transaction — 交易数据。

它还可能包括以下每个可选字段（根据事务的状态）：

* block_hash – 包含交易的区块的哈希值。

* block_number – 区块的序号。

* transaction_index – 交易索引。

* transaction_failure_reason – 交易失败的原因。

## get_transaction_receipt

交易回执除了包含其块信息外，还包含执行信息，如 L1<->L2 交互和消耗的资源，类似于 get_transaction。 要获取交易的收据，请运行以下命令：

```
starknet get_transaction_receipt --hash TRANSACTION_HASH
```

返回结果：

```
{
    "actual_fee": "0x0",
    "block_hash": "0x0",
    "block_number": 0,
    "events": [],
    "execution_resources": {
        "builtin_instance_counter": {
            "pedersen_builtin": 2,
            "range_check_builtin": 10
        },
        "n_memory_holes": 25,
        "n_steps": 309
    },
    "l2_to_l1_messages": [
        {
            "from_address": "0x7dacca7a41e893630664a61f4d8ec05550ca1a212849c62417cb3ecf4bad863",
            "payload": [
                "0x0",
                "0xbc614e",
                "0x3e8"
            ],
            "to_address": "0x9E4c14403d7d9A8A782044E86a93CAE09D7B2ac9"
        }
    ],
    "status": "ACCEPTED_ON_L2",
    "transaction_hash": "0x7797c6673a1a0aeebbcb1c726702e263e5138123124ddef7edd85cd925b11ec",
    "transaction_index": 2
}
```

包含（除了 get_transaction 字段）：

* l2_to_l1_messages – 从 L2 发送到 L1 的消息。

* l1_to_l2_consumed_message – 已接受的信息，是从 L1 发送的。

* execution_resources – 交易执行消耗的资源。

## get_transaction_trace
交易跟踪包含嵌套调用结构中的执行信息； 从外部交易开始的每个调用都包含按时间顺序排列的内部调用列表。 对于每个这样的调用，跟踪包含以下内容：调用者/被调用者地址、选择器、调用数据以及执行信息，例如其返回值、发出的事件和发送的消息。

要获取交易的跟踪信息，请运行以下命令：

```
starknet get_transaction_trace --hash TRANSACTION_HASH
```

结果类似于：
```
{
    "function_invocation": {
        "call_type": "CALL",
        "calldata": [
            "0x1",
            "0x39564c4f6d9f45a963a6dc8cf32737f0d51a08e446304626173fd838bd70e1c",
            "0x15511cc3694f64379908437d6d64458dc76d02482052bfb8a5b33a72c054c77",
            "0x0",
            "0x2",
            "0x2",
            "0xbc614e",
            "0x3e8"
        ],
        "caller_address": "0x0",
        "class_hash": "0x74acbf20e655c80c4fa16e3574489073e0093fd8b386d9614ab2d6cf5b866bf",
        "contract_address": "0x78d796e87cfa496bffad27be9ed42f2709bd6e32a6366f842fdf38664a1412d",
        "entry_point_type": "EXTERNAL",
        "events": [],
        "execution_resources": {
            "builtin_instance_counter": {
                "pedersen_builtin": 2,
                "range_check_builtin": 10
            },
            "n_memory_holes": 25,
            "n_steps": 309
        },
        "internal_calls": [
            ...
        ],
        "messages": [],
        "result": [],
        "selector": "0x15d40a3d6ca2ac30f4031e42be28da9b056fef9bb7357ac5e85627ee876e5ad"
    },
    "signature": [
        "0x6e4606a3c0f3bd0eac37ddfbf2645f62c04474e5eac51a2f6225ee7702996a",
        "0x389d0bae9be71ceb3b6092dda9b76279543bc2bfe271c3d05a812c3dbeffeb7"
    ],
    "validate_invocation": {
        "call_type": "CALL",
        "calldata": [
            "0x1",
            "0x39564c4f6d9f45a963a6dc8cf32737f0d51a08e446304626173fd838bd70e1c",
            "0x15511cc3694f64379908437d6d64458dc76d02482052bfb8a5b33a72c054c77",
            "0x0",
            "0x2",
            "0x2",
            "0xbc614e",
            "0x3e8"
        ],
        "caller_address": "0x0",
        "class_hash": "0x74acbf20e655c80c4fa16e3574489073e0093fd8b386d9614ab2d6cf5b866bf",
        "contract_address": "0x78d796e87cfa496bffad27be9ed42f2709bd6e32a6366f842fdf38664a1412d",
        "entry_point_type": "EXTERNAL",
        "events": [],
        "execution_resources": {
            "builtin_instance_counter": {
                "ecdsa_builtin": 1,
                "range_check_builtin": 2
            },
            "n_memory_holes": 0,
            "n_steps": 89
        },
        "internal_calls": [],
        "messages": [],
        "result": [],
        "selector": "0x162da33a4585851fe8d3af3c2a9c60b557814e221e0d4f30ff0b2189d9c7775"
    }
}
```

## 估算手续费

您可以通过将 ```--estimate_fee``` 标志添加到调用或声明命令来估计给定交易的费用。 这将模拟交易并返回与之相关的估计费用。 您可以在[此处]()阅读有关收费机制的更多信息。 结果以 WEI 和 ETH 表示，如下所示。

请注意，使用 ``--estimate_fee`` 标志，合约的状态不会改变。 例如，以下代码不会影响 B``ALANCE_CONTRACT`` 中存储的余额。

要估算给定交易的费用，请运行以下命令：

```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi contract_abi.json \
    --function increase_balance \
    --inputs 1234 \
    --estimate_fee
```

结果类似于：

```
The estimated fee is: 756000000000000 WEI (0.000756 ETH).
Gas usage: 7560
Gas price: 100000000000 WEI
```

## 模拟交易
```--simulate``` 标志类似于 ```--estimate_fee```，除了它还返回执行事务产生的跟踪。

请注意，使用 ```--simulate``` 标志，与 ```--estimate_fee``` 一样，合约的状态不会改变。 例如，下面的命令不会影响 ```BALANCE_CONTRACT``` 中存储的余额。

要模拟给定事务的执行，请运行以下命令：
```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi contract_abi.json \
    --function increase_balance \
    --inputs 1234 \
    --simulate
```
结果类似于：
```
The estimated fee is: 756000000000000 WEI (0.000756 ETH).
Gas usage: 7560
Gas price: 100000000000 WEI

{
    "function_invocation": {
        "call_type": "CALL",
        "calldata": [
            "0x1",
            "0x39564c4f6d9f45a963a6dc8cf32737f0d51a08e446304626173fd838bd70e1c",
            "0x362398bec32bc0ebb411203221a35a0301193a96f317ebe5e40be9f60d15320",
            "0x0",
            "0x1",
            "0x1",
            "0x4d2"
        ],
        "caller_address": "0x0",
        "class_hash": "0x74acbf20e655c80c4fa16e3574489073e0093fd8b386d9614ab2d6cf5b866bf",
        "contract_address": "0x78d796e87cfa496bffad27be9ed42f2709bd6e32a6366f842fdf38664a1412d",
        "entry_point_type": "EXTERNAL",
        "events": [],
        "execution_resources": {
            "builtin_instance_counter": {
                "range_check_builtin": 2
            },
            "n_memory_holes": 3,
            "n_steps": 206
        },
        "internal_calls": [
            ...
        ],
        "messages": [],
        "result": [],
        "selector": "0x15d40a3d6ca2ac30f4031e42be28da9b056fef9bb7357ac5e85627ee876e5ad"
    },
    "signature": [
        "0x6501fcc88705e138910cf5d8f88cedbc7b6ffde47da5562a94dbc834ce92f4e",
        "0x52371924f8c82221dd895d705eac391f7faefb57b9c55293ede92990355e086"
    ],
    "validate_invocation": {
        "call_type": "CALL",
        "calldata": [
            "0x1",
            "0x39564c4f6d9f45a963a6dc8cf32737f0d51a08e446304626173fd838bd70e1c",
            "0x362398bec32bc0ebb411203221a35a0301193a96f317ebe5e40be9f60d15320",
            "0x0",
            "0x1",
            "0x1",
            "0x4d2"
        ],
        "caller_address": "0x0",
        "class_hash": "0x74acbf20e655c80c4fa16e3574489073e0093fd8b386d9614ab2d6cf5b866bf",
        "contract_address": "0x78d796e87cfa496bffad27be9ed42f2709bd6e32a6366f842fdf38664a1412d",
        "entry_point_type": "EXTERNAL",
        "events": [],
        "execution_resources": {
            "builtin_instance_counter": {
                "ecdsa_builtin": 1,
                "range_check_builtin": 2
            },
            "n_memory_holes": 0,
            "n_steps": 89
        },
        "internal_calls": [],
        "messages": [],
        "result": [],
        "selector": "0x162da33a4585851fe8d3af3c2a9c60b557814e221e0d4f30ff0b2189d9c7775"
    }
}
```

## get_code

一旦交易在链上被接受，您将能够看到您刚刚部署的合约的代码。 输出由字节码列表组成，而不是源代码。 这是因为 StarkNet 网络在编译后得到了合约。

要在特定地址获取合约，请运行以下命令：
```
starknet get_code --contract_address ${CONTRACT_ADDRESS}
```
结果类似于：
```
{
    "abi": [
        {
            "inputs": [
                {
                    "name": "amount",
                    "type": "felt"
                }
            ],
            "name": "increase_balance",
            "outputs": [],
            "type": "function"
        },

        ...

        "0x48127ffb7fff8000",
        "0x48127ffb7fff8000",
        "0x48127ffb7fff8000",
        "0x208b7fff7fff7ffe"
    ]
}
```

## get_class_by_hash

要获得类的哈希值，请运行以下命令：
```
starknet get_class_by_hash --class_hash CLASS_HASH
```
结果类似于：
```
{
    "abi": [
        {
            "inputs": [
                {
                    "name": "amount",
                    "type": "felt"
                }
            ],
            "name": "increase_balance",
            "outputs": [],
            "type": "function"
        },

        ...

    }
}
```

## get_full_contract

要获取特定地址的合约类对象，请运行以下命令：
```
starknet get_full_contract --contract_address ${CONTRACT_ADDRESS}
```
结果类似于：
```
{
    "abi": [
        {
            "inputs": [
                {
                    "name": "amount",
                    "type": "felt"
                }
            ],
            "name": "increase_balance",
            "outputs": [],
            "type": "function"
        },

        ...

    }
}
```

## get_class_hash_at

要获取特定地址的合约哈希，请运行以下命令：
```
starknet get_class_hash_at --contract_address ${CONTRACT_ADDRESS}
```
结果类似于：
```
0x2951dd06d31f492e8ed1e91da115dbcd3ffd7c688f39b4878db99d86995e4c
```

## get_block
您可能希望查询整个区块并检查其中包含的交易，而不是查询特定的合约或交易。 为此，请运行以下命令：
```
starknet get_block --number BLOCK_NUMBER
```
结果类似于：
```
{
    "block_hash": "0x3edbb1517aaf8187b96477a0b42517de577a448666be16c1b85ccd49fbb04b9",
    "block_number": 1,
    "gas_price": "0x174876e800",
    "parent_block_hash": "0x2481e994cd8611047378fffa1454a4a4de1ce420caa28126a546a13443ff840",
    "sequencer_address": "0x310959e4d55cfe4712291a5f9787893fb392d1ffb96905aba549b21e91e9fc9",
    "starknet_version": "0.10.0",
    "state_root": "079354de0075c5c1f2a6af40c7dd70a92dc93c68b54ecc327b61c8426fea177c",
    "status": "ACCEPTED_ON_L2",
    "timestamp": 105,
    "transaction_receipts": [
        {
            "actual_fee": "0x0",
            "events": [],
            "l2_to_l1_messages": [],
            "transaction_hash": "0x50ed2694abe6c3812590cb34f39f367cda58bcfb37c6a61c94da43c60169b07",
            "transaction_index": 0
        }
    ],
    "transactions": [
        {
            "class_hash": "0x4474242ed3116e6fda7f66ab4fbb11e269af89c210dcc1115aad65712b04c95",
            "max_fee": "0x0",
            "nonce": "0x0",
            "sender_address": "0x78d796e87cfa496bffad27be9ed42f2709bd6e32a6366f842fdf38664a1412d",
            "signature": [],
            "transaction_hash": "0x50ed2694abe6c3812590cb34f39f367cda58bcfb37c6a61c94da43c60169b07",
            "type": "DECLARE",
            "version": "0x1"
        }
    ]
}
```

结果包含：

* block_hash – 区块哈希，区块的唯一标识符。

* parent_block_hash – 父区块的区块哈希。

* block_number – 块的序列号，即该区块之前的区块数。

* state_root – 在给定区块之后表示 StarkNet 状态的承诺树的根。

( status – 区块的状态（例如，ACCEPTED_ON_L2，表示该块已创建但尚未在链上被接受）。

* timestamp – 表示此区块创建时间的时间戳。

* transaction_receipts – 关于区块中包含的每笔交易的交易状态和相应的 L1<->L2 交互的信息。

* transaction – 包含在区块中的交易的映射，根据交易哈希。请注意，这些哈希与 transaction_receipts 映射中使用的哈希相同。

要查询的区块，只需传递 ```--number=pending```。要通过哈希查询等待中的区块，请改用 ```--hash```。请注意，最多可以给出这些参数之一。

## get_nonce

您可以使用以下命令检索合约（通常是账户合约）的 nonce：

```
starknet get_nonce --contract_address ${ACCOUNT_ADDRESS}
```

请注意，此命令返回的 ```nonce``` 仅在事务达到等待中（或更高）状态时才会提前。

## get_block_traces

您可能想要查询整个区块并检查其中包含的所有交易的跟踪，而不是查询特定的交易（参见上面的 get_transaction_trace）。 为此，请运行以下命令：

```
starknet get_block_traces --number BLOCK_NUMBER
```

结果包含：

```
{
    "traces": [
        {
            "signature": [
                "0x6e0c9563f403e47703a3d2d788155fc8fc2ae8d0be1b4215a263e0e034f6e2a",
                "0x65299028b39302d5cc12917c63dcc29a15b22f9fca06ae7a58e0fb6dc71f3c"
            ],
            "transaction_hash": "0x1d148f6053e19e6db338101c6d6a9c317ce78e11aaf5ccfb7d7916e941cd375",
            "validate_invocation": {
                "call_type": "CALL",
                "calldata": [
                    "0x4425808c08d99e9088ec93ac678074227dcb78aa12960c0abd927744f03ee78"
                ],
                "caller_address": "0x0",
                "class_hash": "0x29dc39fef1e5e35b506be5fc784b6ee60b862881f7ec61df44e0c4370ac1a67",
                "contract_address": "0x26459ba7cebf9ab0cb59c7490ec8e8a4eb64c46f808f0f666ae7391a128eaf2",
                "entry_point_type": "EXTERNAL",
                "events": [],
                "execution_resources": {
                    "builtin_instance_counter": {
                        "ecdsa_builtin": 1
                    },
                    "n_memory_holes": 0,
                    "n_steps": 73
                },
                "internal_calls": [],
                "messages": [],
                "result": [],
                "selector": "0x289da278a8dc833409cabfdad1581e8e7d40e42dcaed693fa4008dcdb4963b3"
            }
        }
    ]
}
```

## get_state_update
您可以使用以下命令获取特定区块中的状态更改（例如，哪些存储单元已更改）：
```
starknet get_state_update --block_number BLOCK_NUMBER
```
结果包含：
```
{
    "block_hash": "0x703fad93522cd338891ce4e009ca3d71ff742a31fcf87c46d8a5f644e77fbe3",
    "new_root": "0714e01d3a891f71aeaea0dc1af25958fdb71aa0566773e9e208c8ea1adfd3ab",
    "old_root": "02e5810fb9ecbb8d561bcb54b03cfcc3f1b88eda7043148291e2eb974e5ba16c",
    "state_diff": {
        "declared_contracts": [
            "0x4474242ed3116e6fda7f66ab4fbb11e269af89c210dcc1115aad65712b04c95"
        ],
        "deployed_contracts": [
            {
                "address": "0x3a0c6f9001e69edf3c5750475455010ad36eecc6084ebe9ed2c0aa45d029281",
                "class_hash": "0x4474242ed3116e6fda7f66ab4fbb11e269af89c210dcc1115aad65712b04c95"
            }
        ],
        "nonces": {},
        "storage_diffs": {
            "0x17ebb0be75e9b16fc8cc294b6a75194ae562784eb3c8f507dec33c3b28e89e6": [],
            "0x55ce9b2379adaf3820d4877e7162a6e5bd8ac81cf9d00f026973b8647fbb615": [
                {
                    "key": "0x27ae98c4d8558d514da5e6b3eca8e93ec139ddc1388d4e2e80684a991eb4452",
                    "value": "0x2"
                }
            ]
        }
    }
}
```

结果包含：

* block_hash – 区块哈希，区块的唯一标识符。

* new_root – 在给定区块之后表示 StarkNet 状态的承诺树的根。

* old_root – 表示 StarkNet 在给定区块之前的状态的承诺树的根。

* state_diff – 此区块中应用的状态变化，作为地址到新值和/或新合约的映射给出。

要查询最后一个区块，只需删除 ```--block_number``` 参数。 要通过哈希查询块，请改用 ```--block_hash```。 请注意，最多可以给出这些参数之一。

## get_storage_at
除了查询合约的代码之外，您可能还想查询合约在特定键处的存储。 为此，您首先需要了解您感兴趣的密钥。 如您之前所见，StarkNet 引入了一个新的原语，即存储变量。 每个存储变量都映射到一个存储键（一个字段元素）。 要计算此密钥，请运行以下 Python 代码：
```
from starkware.starknet.public.abi import get_storage_var_address

balance_key = get_storage_var_address('balance')
print(f'Balance key: {balance_key}')
```
获取了返回：
```
Balance key: 916907772491729262376534102982219947830828984996257231353398618781993312401
```
现在，您可以使用以下方式查询余额：
```
starknet get_storage_at \
    --contract_address ${CONTRACT_ADDRESS} \
    --key 916907772491729262376534102982219947830828984996257231353398618781993312401
```
使用的相同合约，您应该得到：
```
0x4d2
```

请注意，这与调用 ```get_balance``` 获得的结果相同。

稍后，在用户身份验证部分，您将看到存储变量的泛化，例如，它允许为每个用户设置一个余额变量。 这将需要对上面的代码进行细微的调整，我们将在相关部分进行学习。

## 特定区块的查询

前面提到的一些 CLI 函数有一个附加参数 ```--block_hash```，它将给定的查询应用于特定的区块。 例如，您可能希望在某个特定时间点查询余额变量。

要确定 CLI 函数是否可以作为特定于块的查询执行，只需使用 ```--help``` 参数来查看 ```--block_hash``` 是否是该函数的可选参数的一部分。 如果您不使用 ```--block_hash``` 参数，则查询将应用于最后一个区块。