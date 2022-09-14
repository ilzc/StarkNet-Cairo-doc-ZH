程序输入输出
Cairo 程序可能有一个秘密输入（称为“程序输入”）和一个公共输出（称为“程序输出”）。

假设您想证明您知道哈希函数的原像（即，对于给定的 y，x 满足 hash(x) = y）。 输入将是您的秘密 (x) - 哈希函数的原像。 然后程序将计算这个输入的哈希值，并输出结果。 由于 Cairo 的程序输出是公开的，因此获得证明的每个人都会确信证明者确实知道预期哈希值的原像。

有时，程序输入中的一个值将被复制到程序输出中——例如，考虑语句“我知道第 n 个斐波那契数是 Y”。 在这种情况下，程序输入将是 n，输出将是 n 和 Y —— 因为验证者必须看到 n 是什么。 从输出中删除 n 将导致声明“我知道某个斐波那契数是 Y”的证明。

程序输入
将程序输入添加到 Cairo 程序很容易——您使用程序输入创建一个 json 文件，然后使用 `--program_input` 标志将其传递给 `cairo-run`。 然后，您可以使用变量 program_input 使用提示访问此文件的内容（请记住，提示仅对证明者可见）。

练习

创建以下内容的输入文件 input.json：

```
{
    "secret": 1234
}
```

现在编译并运行以下程序（使用 --program_input=input.json 运行）：

```
func main() {
    %{ memory[ap] = program_input['secret'] %}
    [ap] = [ap], ap++;
    ret;
}
```

程序输出¶
向 Cairo 程序添加输出稍微复杂一些。 首先将以下指令添加到文件顶部（请参阅此处的 [%builtins 指令]()的更多信息）：

```
%builtins output
```

您需要使用不同的布局运行程序才能使用内置输出。 将 `--layout=small` 添加到 cairo-run（在此处查看有关[布局]()的更多信息）。 使用小布局需要步数可以被 512 整除，因此您必须使用 `--steps=512` 运行（这对于小型程序应该足够了）。

`%builtins` 输出指令使函数 main() 获取一个参数并返回一个值。 该参数通常称为 output_ptr ，程序应将其用作指向可以写入其输出的内存块的指针。 main() 应该在写入后返回指针的值，表示输出内存块的结束位置。

以下程序将三个常量值写入输出。

```
%builtins output

func main(output_ptr: felt*) -> (output_ptr: felt*) {
    [ap] = 100;
    [ap] = [output_ptr], ap++;

    [ap] = 200;
    [ap] = [output_ptr + 1], ap++;

    [ap] = 300;
    [ap] = [output_ptr + 2], ap++;

    // Return the new value of output_ptr, which was advanced
    // by 3.
    [ap] = output_ptr + 3, ap++;
    ret;
}
```

注意 `output_ptr` 是输出指针的值，而 `[output_ptr]` 是它指向的值。 另请注意，`[output_ptr] = 100` 不是有效的 Cairo 指令，因此我们将其拆分为两条指令（有关指令列表，请参阅[基本指令]()）。