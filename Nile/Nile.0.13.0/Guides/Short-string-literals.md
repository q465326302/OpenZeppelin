# Short String Literals
根据[cairo-lang](https://www.cairo-lang.org/docs/how_cairo_works/consts.html#short-string-literals)文档：

“短字符串是长度最多为31个字符的字符串，因此可以适应单个字段元素”。

## Nile集成
在Nile中，合约调用参数（calldata）既不是整数也不是十六进制数，都被视为短字符串，并自动转换为相应的felt表示。

因此，您可以从CLI运行以下命令:
```
nile deploy MyToken 'MyToken name' 'MyToken symbol' ...
```

这等同于直接传递felt表示，如下所示：
```
nile deploy MyToken 0x4d79546f6b656e206e616d65 0x4d79546f6b656e2073796d626f6c ...
```

> NOTE
如果您想将代币名称作为十六进制或整数传递，您需要直接提供felt表示，因为这些值不会被解释为短字符串。您可以使用str_to_felt工具。
```
>>> from nile.utils import str_to_felt
>>>
>>> str_to_felt('any string')
460107418789485453340263
```