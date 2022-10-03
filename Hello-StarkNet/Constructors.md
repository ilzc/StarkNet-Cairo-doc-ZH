# 构造函数

一个合约需要在被公开使用前进行初始化。例如，一个人可能想要指定一个合约所有者，它可以执行其他用户无法执行的某些操作。在构造函数中，可通过设置存储变量的方式记录合约所有者。合约构造函数是使用 ```@constructor``` 装饰器定义的，它的名字必须是 ```constructor```。构造函数的语义与任何其他外部函数的语义相似，只是构造函数在合约部署期间保证运行，并且在合约部署后不能再次调用。

例如，我们可以如下定义一个 ownable 的合约：

```
%lang starknet

from starkware.cairo.common.cairo_builtins import HashBuiltin

// 定义一个存储变量来记录 owner 地址
@storage_var
func owner() -> (owner_address: felt) {
}

@constructor
func constructor{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}(owner_address: felt) {
    owner.write(value=owner_address);
    return ();
}

@view
func get_owner{
    syscall_ptr: felt*,
    pedersen_ptr: HashBuiltin*,
    range_check_ptr,
}() -> (address: felt) {
    let (address) = owner.read();
    return (address=address);
}
```

将合约保存为 ownable.cairo ，然后编译并声明它。

```
starknet-compile ownable.cairo \
    --output ownable_compiled.json \
    --abi ownable_abi.json
starknet declare --contract ownable_compiled.json
```

编译完成后，合约就可以部署了。 部署合约时，使用 --inputs 参数传递参数。 输入的数量必须与构造函数的签名相匹配。 否则，部署的交易将失败。

```
starknet deploy --contract ownable_compiled.json --inputs 123 --no_wallet
```

合约部署时，合约地址、合约类哈希和构造函数调用数据都包含在链上数据中。