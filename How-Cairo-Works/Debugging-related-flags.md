# Debug用到的一些 flags

以下是 `cairo-run` 命令的一些 flags ，您可以使用它们来调试程序中的错误。 除了这些 flags 之外，您还可以随时添加带有打印语句的[提示(hint)]()，以输出某个标识符或内存单元的值。

## [--print_info](#print-info)
`cairo-run` 命令的 flags 能够打印包含下列一些信息：

1.执行的步骤数。

2.已用的内存单元数。

3.执行完成后寄存器中的数值。

4.段重定位表。

您可能留意到输出的格式是 x:y （例如. 4:2），这些事重定位值，跳转到 [段]() 获得更多信息。

## [--print_memory](#print-memory)
打印程序在执行中的内存地址和对应的值。

如果您发现在对应内存位置出现错误，检查该内存地址或者相邻的内存位置可能会有帮助。

## [--steps](#steps)
您能够指定 `cairo-run` 运行的步骤数。这是一个可选的选项，如果您没有指定，那么 `cairo-run` 将会运行到程序结束的位置（在 main() 方法的 ret 指令之后）。

## [--no_end](#no-end)
如果您指定了运行的步骤数，那么程序在指定步骤内不会结束， `cairo-run` 将会抛出错误（不能到达程序结束位置）。您能够使用 --no_end 执行 `cairo-run` 来忽略这个错误。通常情况下，使用这个 flag 能够在程序运行指定数量的步骤后检查内存和寄存器的值。

## [--debug_error](#debug-error)
发生错误时，默认情况下不打印内存和信息部分。无论如何，您都可以使用 --debug_error 来打印它们。通过检查内存和信息部分的值，您可能能够更好地了解错误的来源。

## [--profile_output profile.pb.gz](#more)
可以使用 pprof 查看的配置文件结果文件。 如果你已经安装了，你可以使用以下命令查看结果：

```
go tool pprof --web profile.pb.gz
```

或者，您可以使用 golang docker 镜像（需要先安装 docker）：

```
docker run --net=host -v$PWD:/tmp starkwarelibs/pprof profile.pb.gz
```

或者使用这个 docker 文件手动构建 docker 镜像：

```
FROM golang
RUN apt update
RUN apt install -y graphviz
WORKDIR /tmp
ENTRYPOINT ["go", "tool", "pprof", "--http", "localhost:8080", "--no_browser"]
```
