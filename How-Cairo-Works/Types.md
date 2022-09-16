# 类型

## [元组](#tuple)

元组类型的定义类似于元组表达式。 例如，给定两个类型 `a` 和 `b`，类型 `(a, b)` 表示一个元组，该元组由一个类型为 `a` 的元素和一个类型为 `b` 的元素组成。 例如`（felt，felt）`可以用来表示（二维）点。

Cairo 还支持命名元组，例如 (x: Felt, y: Felt) 表示一个类似于 (felt, felt) 的元组，只是这两个元素分别命名为 x 和 y。


## [用户定义的类型别名](#alias)
您可以为类型指定一个新别名，如下所示：

`using Point = (x: felt, y: felt);`
请注意，在这种情况下，Point 不是一种新类型——它只是 (x: felt, y: felt) 的别名。 您可以使用 Point 作为此类型的别名。

例如，您可以替换

`local pt: (x: felt, y: felt) = (x=2, y=3);`
成为：

`local pt: Point = (x=2, y=3);`
