# 编写 StarkNet 合约

为了学习本教程，您应该对编写 Cairo 代码有基本的了解。 例如，您可以阅读“Hello, Cairo”教程的前几页。 您还应该设置您的环境并确保您安装的 Cairo 版本至少为 0.10.0（您可以通过运行 cairo-compile --version 检查您的版本）。

## 动手写第一个合约
让我们从以下 StarkNet 合约开始：

```
// 声明这个文件是一个 StarkNet 合约
%lang starknet

from starkware.cairo.common.cairo_builtins import HashBuiltin

// 定义一个存储变量
@storage_var
func balance() -> (res: felt) {
}

// 使用指定的值增加余额
@external
func increase_balance{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(amount: felt) {
    let (res) = balance.read();
    balance.write(res + amount);
    return ();
}

// 返回当前余额
@view
func get_balance{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}() -> (res: felt) {
    let (res) = balance.read();
    return (res=res);
}
```

第一行，`%lang starknet` 声明这个文件应该被读取为一个 StarkNet 合约文件，而不是一个常规的 Cairo 程序文件。 尝试使用 `cairo-compile` 编译此文件将导致编译错误。 编译 StarkNet 合约应该使用 `starknet-compile` 命令完成，我们将在下面看到。

接下来，我们有一个导入语句。 如果您不熟悉这种类型的语句，请参阅“Hello, Cairo”教程。 请注意，您不需要在 StarkNet 合约中显式使用 `%builtins `指令。

我们在代码中看到的第一个新原语是`@storage_var`。 与无状态的 Cairo 程序不同，StarkNet 合约有一个状态，称为“合约的存储”。 在此类合约上调用的交易可能会以合约定义的方式修改此状态。

`@storage_var` 装饰器声明了一个变量，该变量将作为存储的一部分。在我们的例子中，这个变量由一个 felt 组成，称为 `balance` 。要使用这个变量，我们将使用由 `@storage_var` 装饰器自动创建的 `balance.read()` 和 `balance.write()` 函数。部署合约时，其所有存储单元都初始化为零。特别是，所有存储变量最初都是零。

StarkNet 合约没有 `main()` 函数。相反，每个函数都可以被注释为一个外部函数（使用 `@external` 装饰器）。 StarkNet 的用户和其他合约可以调用外部函数（请参阅[调用合约]()）。

在我们的例子中，合约有两个外部函数：`increase_balance` 从存储中读取余额的当前值，将给定的金额添加到其中，并将新值写回存储中。 `get_balance` 只是读取余额并返回其值。

`@view` 装饰器与 `@external` 装饰器相同。唯一的区别是该方法被注释为一个只查询状态而不能修改写入的方法。请注意，在当前版本中，编译器不强制执行此操作。

考虑三个隐式参数：`syscall_ptr`、`pedersen_ptr` 和 `range_check_ptr`：

1. 您应该熟悉 `pedersen_ptr`，它允许计算 Pedersen 散列函数，以及 `range_check_ptr`，它允许比较整数。 但是合约似乎没有使用任何哈希函数或整数比较，那么为什么需要它们呢？ 原因是存储变量需要这些隐式参数才能计算该变量的实际内存地址。 这在诸如 balance 之类的简单变量中可能不需要，但是使用映射（请参阅[存储映射]()）计算 Pedersen 哈希是 `read()` 和 `write()` 所做的一部分。

2. `syscall_ptr` 是 StarkNet 合约独有的新原语（它在 Cairo 中不存在）。 `syscall_ptr` 允许代码调用系统调用。 它也是 read() 和 write() 的隐式参数（在这种情况下是必需的，因为存储访问是使用系统调用完成的）。

## 不使用提示进行编程
如果你熟悉 Cairo 的话，你可能熟悉提示。不幸的是（或幸运的事，取决于您怎么考虑）， 在 StarkNet 中使用提示是不可能的。这是因为合约的作者、调用函数的用户和运行它的操作者可能是不同的实体：

1. 出于安全考虑，操作员无法运行任意 Python 代码。
2. 用户将无法验证操作员是否运行了合约作者提供的提示。
3. 不可能证明非确定性代码失败，因为您应该证明您执行了提示或证明对于任何提示代码都会失败。

为了效率，标准库函数仍然使用提示，通过白名单机制（如果一个函数同意运行它，当它知道它可以成功运行它的提示时，它会被操作员列入白名单。它不必与库函数的健全性问题有关，应单独验证）。这意味着在编写 StarkNet 合约时，并非所有 Cairo 库函数都可以使用。有关列入白名单的库函数的列表，请参见[此处](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/starknet/security/starknet_common.cairo)。

## 编译合约
创建一个名为 contract.cairo 的文件并将合约代码复制到其中。

运行以下命令来编译你的合约：

```
starknet-compile contract.cairo \
    --output contract_compiled.json \
    --abi contract_abi.json
```
如上所述，我们无法使用 `cairo-compile` 编译 StarkNet 合约，我们需要使用 `starknet-compile` 来代替。

## 合约的ABI
让我们检查在合约编译期间创建的文件 contract_abi.json：

```
[
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
    {
        "inputs": [],
        "name": "get_balance",
        "outputs": [
            {
                "name": "res",
                "type": "felt"
            }
        ],
        "stateMutability": "view",
        "type": "function"
    }
]
```

ABI 文件包含所有可调用函数及其预期输入的参数列表。

## 在 StarkNet 测试网上声明合约
为了令 CLI 在 StarkNet 测试网环境下工作，您应该在每次使用时传递 `--network=alpha-goerli`，或者设置 `STARKNET_NETWORK `环境变量，如下所示：

```
export STARKNET_NETWORK=alpha-goerli
```

与以太坊不同，StarkNet 区分合约类和合约实例。 合约类代表合约的代码（但没有状态），而合约实例代表类的特定实例，具有自己的状态。

运行以下命令在 StarkNet 测试网上声明您的合约类：

```
starknet declare --contract contract_compiled.json
```

输出应如下所示：
```
Declare transaction was sent.
Contract class hash: 0x1e2208b571b2cb68908f37a196ed5e391c8933a6db23bb3939acedee40d9b8a
Transaction hash: 0x762e166dd3326b2e263eb5bcfdccd225dc88e067fdf7c92cf8ce5e4ea01f9f1
```

你可以在这里看到你的新合约的定义的类对应的哈希值。 为了使用 deploy 系统调用部署合约实例，您将需要此哈希。

## 部署合约到 StarkNet 测试网
重要提示：alpha 版本是一个实验版本。 较新的版本可能需要重置网络状态（导致已部署的合约会被删除）。

运行以下命令在 StarkNet 测试网上部署您的合约（将 `$CLASS_HASH` 替换为您从 starknet 声明中获得的类哈希值）：
```
starknet deploy --class_hash $CLASS_HASH
```
输出应如下所示：
```
Invoke transaction for contract deployment was sent.
Contract address: 0x039564c4f6d9f45a963a6dc8cf32737f0d51a08e446304626173fd838bd70e1c
Transaction hash: 0x125e4bc5251af8ee2664ea0d1495b36c593f25f78f1a78f637a3f7aafa9e22
```
您可以在此处查看新合同的地址。 你需要这个地址来与合约交互。

设置以下环境变量：
合约的部署地址
```
export CONTRACT_ADDRESS="<address of the previous contract>"
```

## 和合约交互
运行以下命令来调用 `increase_balance()`：
```
starknet invoke \
    --address ${CONTRACT_ADDRESS} \
    --abi contract_abi.json \
    --function increase_balance \
    --inputs 1234
```
结果如下所示：
```
Invoke transaction was sent.
Contract address: 0x039564c4f6d9f45a963a6dc8cf32737f0d51a08e446304626173fd838bd70e1c
Transaction hash: 0x69d743891f69d758928e163eff1e3d7256752f549f134974d4aa8d26d5d7da8
```
注意：由于 StarkNet 中使用手续费，通过函数调用与合约的每次交互都必须使用帐户完成。 要设置帐户，请参阅设置 StarkNet 帐户。

以下命令允许您根据获得的交易哈希查询交易状态（这里您必须将 `TRANSACTION_HASH` 替换为由 `starknet invoke` 打印的交易哈希）：

```
starknet tx_status --hash TRANSACTION_HASH
```
有如下输出:
```
{
    "block_hash": "0x0",
    "tx_status": "ACCEPTED_ON_L2"
}
```

可能的状态是：

* **NOT_RECEIVED**：交易尚未收到（即未写入存储）。

* **RECEIVED**：交易被定序器接收。

* **PENDING**：交易通过验证，进入待处理区块。

* **REJECTED**：交易未通过验证，因此被跳过。

* **ACCEPTED_ON_L2**：交易通过验证，进入实际创建的区块。

* **ACCEPTED_ON_L1**：交易在链上被接受。

## 查询余额
使用以下命令查询当前余额：

```
starknet call \
    --address ${CONTRACT_ADDRESS} \
    --abi contract_abi.json \
    --function get_balance
```
有如下输出:
```
1234
```
请注意，要查看最新余额，您应该等到 `increase_balance` 交易状态至少为 `ACCEPTED_ON_L2`（即 `ACCEPTED_ON_L2` 或 `ACCEPTED_ON_L1`）。 否则，您将在 `increase_balance` 交易执行之前看到余额（即 0）。

在下一节中，我们将描述用于查询 StarkNet 状态的其他 CLI 函数。 请注意，虽然部署和调用会影响 StarkNet 的状态，但所有其他功能都是只读的。 特别是，在可能会改变状态的函数上使用 call 而不是 invoke .

