# 特点


# 环境

- 输入`erl`或者`erl.exe`就可以进入erlang环境。
- 随便输入什么算式，需要以`.`结尾才能被执行
- 使用`Ctrl+C`再按`a`或者`halt()`退出erlang环境

# 模块&函数
下面是一个简单的erlang程序，保存为`fac.erl`
```erlang
-module(mymodule).
-export([fac/1]).

fac(1) ->
    1;
fac(N) ->
    N * fac(N - 1).
```

程序使用递归实现了一个计算阶乘的逻辑（erlang没有循环）

第一行定义了一个名叫`momodule`的模块
第二行定义了想外部暴露的函数，这里暴露了`fac`函数，函数的参数个数为`1`个。

erlang支持函数重载，可以使用不同个数的参数：
```erlang
fac(1,1) -> 1;
```

