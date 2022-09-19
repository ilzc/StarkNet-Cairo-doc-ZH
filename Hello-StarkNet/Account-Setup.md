# 设置 StarkNet 帐户

## 安装
按照[设置环境]()中的 cairo-lang 包的安装说明进行操作。

## 设置网络
在本教程中，我们将使用 StarkNet CLI（命令行界面）与 StarkNet 进行交互。 为了指示 CLI 与 StarkNet 测试网一起工作，您可以将 `--network=alpha-goerli` 标志添加到每个命令，或者简单地设置 `STARKNET_NETWORK` 环境变量，如下所示：

```
export STARKNET_NETWORK=alpha-goerli
```

## 选择钱包提供商
与区分外部拥有账户 (EOA) 和合约的以太坊不同，StarkNet 没有这种账户。 相反，账户由定义账户逻辑的已部署合约来表示——最值得注意的是谁可以从中发出交易的签名方案。

要与 StarkNet 交互，您需要部署一个账户合约。 在本教程中，我们将使用 OpenZeppelin 的 EOA 合约标准的略微修改版本（目前，签名的计算方式不同）。 设置 STARKNET_WALLET 环境变量如下：

```
export STARKNET_WALLET=starkware.starknet.wallets.open_zeppelin.OpenZeppelinAccount
```

## 创建一个账户
运行下面的命令来创建账户
```
starknet deploy_account
```
输出应类似于：

```
Sent deploy account contract transaction.

NOTE: This is a modified version of the OpenZeppelin account contract. The signature is computed
differently.

Contract address: ...
Public key: ...
Transaction hash: ...
```

如果您想维护多个帐户，您还可以使用 `--account=my_account` 为您的帐户指定一个名称。如果未指定，则使用默认帐户（名为 `__default__`）。

`STARKNET_WALLET` 环境变量指示 StarkNet CLI 在 `starknet invoke` 和 `starknet call` 命令中使用您的帐户。如果你想直接调用合约，而不通过你的账户合约，你可以将 `--no_wallet` 参数传递给 CLI，它会覆盖 `STARKNET_WALLET` 变量。

警告：使用作为 `cairo-lang` 包 (`starkware.starknet.wallets...`) 一部分的内置钱包提供程序是不安全的（例如，私钥可能未加密并且在您的主目录中没有备份）。仅当您不太担心失去对帐户的访问权限时（例如，出于测试目的），才应使用它们。此外，它们不是使用代理模式部署的，因此它们无法升级，并且可能在 StarkNet 的未来版本中停止工作。

## 发送 Goerli 测试网 ETH 到账号
为了在 StarkNet 上执行交易，您需要在您的 L2 账户中持有 ETH（用于支付交易费用）。

您可以通过以下方式获取 L2 ETH：

1. 使用 StarkNet Faucet 将少量 ETH 直接存入您刚刚创建的账户。 这对于简单的交易应该足够了。

2. 使用 StarkGate – StarkNet L2 桥接器（L1 合约/Web 应用程序）将您现有的 Goerli L1 ETH 转入和转出 L2 账户。