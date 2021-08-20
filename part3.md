<img class="elixirtip-img" src="assets/images/parts/elixir_tips_3.jpg"/>

## 1. 使用函数作为断言(Guard Clauses)

在 Elixir 中并不能直接将函数用作断言。这意味着， `when` 不接受返回 `Boolean` 的函数作为条件。思考下面几行代码…

```elixir
defmodule Hello do
  def hello(name, age) when is_kid(age) do
    IO.puts "Hello Kid #{name}"
  end
  def hello(name, age) when is_adult(age) do
    IO.puts "Hello Mister #{name}"
  end
  def is_kid age do
    age < 12
  end
  def is_adult age do
    age > 18
  end
end
```

这里我定义了一个名为`Hello`的模块和一个名为`hello`的方法，这个方法的入参为 `name` 和 `age`。 这里我们根据不同的年龄输出不同的内容。如果你编译上面的代码，你会发现编译报错：

```text
** (CompileError) hello.ex:2: cannot invoke local is_kid/1 inside guard
    hello.ex:2: (module)
```

这是因为 **when** 不接受函数作为断言。因此我们必须先将他们转换为宏(`macros`)，就像这样：

```elixir
defmodule MyGuards do

  defmacro is_kid age do
    quote do: unquote(age) < 12
  end

  defmacro is_adult age do
    quote do: unquote(age) > 18
  end

end
# 这里的模块顺序很重要(必须先定义MyGuards，因为下面的模块会用到).....
defmodule Hello do

  import MyGuards

  def hello(name, age) when is_kid(age) do
    IO.puts "Hello Kid #{name}"
  end

  def hello(name, age) when is_adult(age) do
    IO.puts "Hello Mister #{name}"
  end
  
  def hello(name, age) do
    IO.puts "Hello Youth #{name}"
  end

end
```

观察上面的代码，我们将所有的断言放到了 `MyGuards` 中，并且确保在 `Hello` 模块前面，这样，编译器会优先编译 Hello 模块需要的断言。编制之后执行，你会看到你想要的结果：

```elixir
iex> Hello.hello "blackode", 21
Hello Mister blackode
:ok
iex> Hello.hello "blackode", 11
Hello Kid blackode
:ok
```

Elixir v1.6 之后，你可以使用 [defguard/1](https://hexdocs.pm/elixir/Kernel.html#defguard/1)。

`defguard` 也是一个宏。你可以用 `defguardp` 定义自己的断言。希望你能明白这个点。
思考下面的例子。

**注意**: `defguard` 和 `defguardp` 想其他宏一样，必须定义在模块内部。如果在定义的时候存在不能放入 `when` 断言的东西，会引发编译时错误。

假设，你需要判断一个变量等于`3`或者`5`，你可以这样做。

```elixir
defmodule Number.Guards do
  defguard is_three_or_five(number) when (number===3) or (number===5)
end
```

## 使用

```elixir
import Number.Guards
defmodule Hello do
  def check_favorite_number(num) when is_three_or_five(num) do
    IO.puts "The given #{num} is on of my favourite numbers"
  end
  def check_favorite_number(_num), do: IO.puts "Not my favorite number"
end
```

你还可以在代码中使用它们，因为它们会产生返回值是 boolean 类型。

```elixir
iex> import Number.Guards
Number.Guards

iex> is_three_or_five(5)
true

iex> is_three_or_five(3)
true

iex> is_three_or_five(1)
false
```

可以看看下面的运行截图

![ScreenShot Defguard Execution](.gitbook/assets/defguard%20%281%29.png)

## 2. 判断字符串子集

使用 `=~` 操作符，我们可以判断 **右边** 的字符串是否是 **左边** 的字符串的子集。

```elixir
iex> "blackode" =~ "kode" 
true  
iex> "blackode" =~ "medium" 
false  
iex> "blackode" =~ "" 
true
```

## 3. 判断模块是否加重

有些情况下，我们在调用模块方法前必须先确保该模块已经加载。

```text
Code.ensure_loaded? <Module>
```

```elixir
iex> Code.ensure_loaded? :kernel
true
iex> Code.ensure_loaded :kernel
{:module, :kernel}
```

同样，`Code` 还提供了 `ensure_compiled` 方法来判断模块是否编译

## 4. Binary 转换为首字母大写的 Atom

Elixir 对于模块名的规定比较特殊。模块名的规则：首字母为 _**大写 ASCII 字母**_ 后面跟随任意数量的 _小写_ or _大写 ASCII 字母_, _数字_，或者 _下划线_。

实际上，模块名的前缀都包含有 `Elixir.`， 所以 `defmodule Blackode` 中的 `Blackode` 相当于 `:"Elixir.Blackode"`

当我们使用 `String.to_atom "Blackode"` 时只能转换成 `:Blackode` ，但实际上我们需要将 “**Blackode” 变成 Blackode**。因此我们需要使用 `Module.concat`

```elixir
iex(2)> String.to_atom "Blackode"
:Blackode
iex(3)> Module.concat Elixir,"Blackode"
Blackode
```

在运行命令行程序的时候，无论你输入什么，都是以 **binary** 类型出现。 所以，有时候你会用到上面的技巧来转换它们。

## 5. 模式匹配 与 解构函数(destructure)

我们都知道 `=` 进行模式匹配的话，左右两边必须相等。 因此我们这样用的时候 `[a, b, c] = [1, 2, 3, 4]` 会抛出 `MatchError` 异常。

```elixir
iex(11)> [a, b, c] = [1, 2, 3, 4]
** (MatchError) no match of right hand side value: [1, 2, 3, 4]
```

我们可以试试用 `destructure/2` 完成这个工作。

```elixir
iex(1)> destructure [a, b, c], [1, 2, 3, 4]
[1, 2, 3]
iex(2)> {a, b, c}
{1, 2, 3}
```

If the left side is having more entries than in right side, it assigns the `nil` value for remaining entries..

```elixir
iex> destructure([a, b, c], [1])
iex> {a, b, c} 
{1, nil, nil}
```

## 6. 使用 `inspect` 的 `:label` 选项来装饰打印数据

在我们使用 `inspect` 打印数据时，我们可以通过 `label` 选项来装饰一下打印的输出， 设置的 `label` 字符串将会放在打印数据的最前面。

```elixir
iex(1)> IO.inspect [1, 2, 3], label: "the list "
the list : [1, 2, 3]
[1, 2, 3]
```

你仔细观察会发现，`IO.inspect` 返回的数据跟原来的是一样的。 所以，你可以像下面这样，在 `|>` 之间使用快速查看数据：

```elixir
[1, 2, 3] 
|> IO.inspect(label: "before change") 
|> Enum.map(&(&1 * 2)) 
|> IO.inspect(label: "after change") 
|> length
```

你能看到下面这样的 `输出`

```elixir
before change: [1, 2, 3]
after change: [2, 4, 6]
3
```

## 7. 在管道(pipe)中使用匿名函数

这里用两种方法，其中一种是像下面这样直接使用 `&` ：

```elixir
[1, 2, 3, 4, 5]
|> length()
|> (&(&1*&1)).()
```

下面这个方法比较诡异。尽管如此，我们可以通过指定匿名函数的名称来使用该函数的引用。

```elixir
square = & &1 * &1
[1, 2, 3, 4, 5]
|> length()
|> square.()
```

第二种写法会优于第一种写法。另外你也可以通过 `fn` 来定义匿名函数。

## 8. 快速查看字符对应的数字编码 — ?

我们可以用 `?` 操作符去查看字符对应的数字编码。

```elixir
iex> ?a
97
iex> ?#
35
```

以下两个技巧对初学者非常有用

## 9. Lists减法

我们可以对列表执行减法，以删除列表中的元素。

```elixir
iex> [1, 2, 3, 4.5] -- [1, 2]
[3, 4.5]
iex> [1, 2, 3, 4.5, 1] -- [1]  
[2, 3, 4.5, 1]
iex> [1, 2, 3, 4.5, 1] -- [1, 1]
[2, 3, 4.5]
iex> [1, 2, 3, 4.5] -- [6]
[1, 2, 3, 4.5]
```

我们也可以对字符列表执行相同的操作：

```elixir
iex(12)> 'blackode' -- 'ode'
'black'
iex(13)> 'blackode' -- 'z'    
'blackode'
```

如果列表中不存在要减去的元素，则它只返回原来的列表。

## 10. 在 IEx 中使用以前的运行结果

在 `iex` 环境下 , 你会发现每次执行完一个语句，数字都会递增1，就像这样： `iex(2)>` `iex(3)>`

IEx 默认加载了`v/1`函数，通过这些数字，我们可以使用该函数可以获取对应数字下的运行结果。

```elixir
iex(1)> list = [1, 2, 3, 4, 5]
[1, 2, 3, 4, 5]
iex(2)> double_list = Enum.map(list, &(&1*2))
[2, 4, 6, 8, 10]
iex(3)> v 1
[1, 2, 3, 4, 5]
iex(4)> v(1) ++ v(2)
[1, 2, 3, 4, 5, 2, 4, 6, 8, 10]
```

[前一篇](part2.md) [下一篇](part4.md)
