# 导入包
在 Cairo，与其他编程语言一样，将整个程序保存在一个源文件中是不可取且不切实际的。为了避免这种情况并促进模块化，Cairo 允许从另一个文件导入模块，语法如下：

```
from a.b import c as d
```

这将搜索 Cairo 模块 `a.b`，并从中导入 `c`，将其绑定到该模块中的名称 `d`。 `as` 子句是可选的，没有它，`c` 将简单地绑定到本地名称 `c`。可以使用以下语法从同一个 Cairo 文件中导入多个名字：

```
from a.b import c as d, e, f
```

## 导入搜索路径

当 Cairo 在上面的示例中查找模块 `a.b` 时，它将在已配置搜索的路径中搜索文件 `a/b.cairo`。搜索的路径取自一个以冒号分隔的列表，可以通过两种方式设置：

1. Cairo 编译器的 `--cairo_path` 参数。

2. 环境变量 `CAIRO_PATH`。

例如，为了将 `/home/cairo_libs` 和 `/tmp/cairo_libs` 添加到 Cairo 编译器使用的路径列表中，可以运行以下任一行：

```
cairo-compile --cairo_path="/home/cairo_libs:/tmp/cairo_libs" ...

CAIRO_PATH="/home/cairo_libs:/tmp/cairo_libs" cairo-compile ...
```

此外，Cairo 会搜索当前目录和相对于编译器路径的标准库目录。

Cairo 编译器将自动检测循环导入和在单个 Cairo 文件中引用相同名称的多个导入。