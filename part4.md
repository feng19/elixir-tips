<img class="elixirtip-img" src="assets/images/parts/elixir_tips_4.jpg"/>

## 1. 一次运行多个 Mix 任务

```elixir
mix do deps.get,compile
```

运行多个任务的时候，可以通过逗号 `,` 来分割他们。

不过你也可以在你的 mix 项目的文件中  `mix.exs` 创建别名。

当你使用 `mix` 创建的项目一般是下面这样的：

```elixir
def project do
    [app: :project_name,
     version: "0.1.0",
     elixir: "~> 1.4-rc",
     build_embedded: Mix.env == :prod,
     start_permanent: Mix.env == :prod,
     deps: deps()]
  end
```

你可以修改其中一些字段。

像下面这样增加 `aliases` 字段。

```elixir
[
 aliases: aliases()
]
```

当你把上面的拷贝到你的文件中是，你是在 `list` 的结构里面修改，记得给上一行的最后加上 `,` 逗号。

`aliases()` 函数应该返回 `key-value` 列表。

```elixir
defp aliases do
  [
    "ecto.setup": ["ecto.create", "ecto.migrate", "ecto.seed"]
  ]
end
```

完成上面的步骤之后，每当你执行 `mix ecto.setup` 时，这三个 `ecto.create`， `ecto.migrate` 和 `ecto.seed` 就会一个一个接着被执行。

你也可以直接将列表加到 `project` 函数内，并不一定非要放到函数内，就像下面这样：

```elixir
def project do
    [app: :project_name,
     version: "0.1.0",
     aliases: ["ecto.setup": ["ecto.create", "ecto.migrate", "ecto.seed"]]      
.....
  end
```

## 2. 获取文档

Elixir 以 `bytecode` 方式存储文档。你可以通过 `Code.fetch_docs/1` 函数获取文档。这就是说，文档仅仅在需要的时候才回去从硬盘中获取，并不是跟随模块加载一起加载到 vm 中的。

假设你在 **IEx** 中定义了一个模块，此模块仅仅存在于内存中，并不能获取此模块的文档，因为 `bytecode` 并未写入到硬盘中。

让我们开始吧，将下面的代码复制到 `test.ex` 文件中：

```elixir
defmodule Test do
  @moduledoc """
  This is the test module docs
  """

  @doc """
  This is the documentation of hello function
  """
  def hello do
    IO.puts "hello"
  end
end
```

在同一个目录下运行下面这条命令语句：

```elixir
$ iex test.ex
```

但是这样你只能调用函数，并不能获取到文档。

```elixir
iex> Test.hello
hello
:ok
```

这意味着，代码已经加载了，但是文档并未加载到内存中；所以你无法获取到文档，你可以通过下面的函数试试：

```elixir
iex> Code.fetch_docs Test
{:error, :module_not_found}
```

当你尝试范围你创建模块的文档时候，你会发现返回了 `nil` 值。这是因为 `bytecode` 并不存在于硬盘中。简单可以理解为 `beam` 文件不存在导致的，那让我们换种方式看看……

退出`iex`控制台然后重新回到`shell`控制台，然后执行下面的命令：

```elixir
$ elixirc test.ex
```

运行过后，你可以在当前目录发现一个文件： `Elixir.Test.beam` 。 现在模块`Test`的 `bytecode` 就能加载到内存中了，然后你就能用下面的函数访问到文档了：

```elixir
$ iex
iex> Code.fetch_docs Test
{:docs_v1, 2, :elixir, "text/markdown",
 %{"en" => "This is the test module docs\n"}, %{},
 [
   {{:function, :hello, 0}, 6, ["hello()"],
    %{"en" => "This is the documentation of hello function\n"}, %{}}
 ]}
```

查看链接了解更多详情： [链接](https://hexdocs.pm/elixir/Code.html#get_docs/2)

## 3. 详细的测试报告

当你运行 `mix test` 时，所有的测试都会执行，最后打印总的耗时时间。 然而，如果你还想看到更加详尽的信息可以在命令行的最后增加  `--trace` 选项，看下面的例子：

```elixir
mix test --trace
```

定义 `test "test_string"` 中的 `test_string` 就是这个 test 的名字，上面的结果会打印出所有的名字以及每个测试的消耗用时。

## 4. 通过 Elixir 的 宏(Macro) 动态定义函数名

```elixir
defmacro gen_function(fun_name) do
  quote do 
    def unquote(:"#{fun_name}")() do
      # your code...
    end
  end
end
```

简单来说，函数的名字必须是一个 **原子(atom)** 而不是字符串。

## **5. 在 Elixir 中运行 Shell 命令**

```elixir
System.cmd(command, args, options \\ [])
```

参数解析：

* **command** 可执行文件/程序，并且需要在 PATH 中能查找到的，除非提供绝对路径。
* **args** 必须是一个列表，元素需为字符串，可执行文件/程序会收到这些参数。

### 示例

```elixir
iex> System.cmd "echo", ["hello"]
    {"hello\n", 0}
```

```elixir
iex> System.cmd "echo", ["hello"], into: []
    {["hello\n"], 0}
```

可以在 `iex` 中输入 `h System.cmd` 查看帮组文档

可以去查看 `System` 的文档，或者 [Erlang os Module](http://www.erlang.org/doc/man/os.html)

## 6. 打印 列表(List) 时不转换 ASCII 编码

如果列表中所有的数字都在 **ASCII** 编码表范围内的话，输出时会自动将数字对应的字符打印出来，如下面的例子：

```elixir
iex> IO.inspect [97, 98]
'ab'
'ab'
```

字符  `a` 的编码是 `97` ，字符 `b` 是 `98` ，因此就以 `字符列表` 形式输出。尽管如此，你可以给 `IO.inspect` 设置 `char_lists: :as_list` 参数来避免。

```elixir
iex> IO.inspect [97, 98], charlists: :as_lists
[97, 98]
'ab'
```

打开 `iex` 然后输入 `h Inspect.Opts`, 你会发现 Elixir 其它类型的数据时依然有效，特别是 **structs** 和 **binaries**。

## 7. 获取当前文件名/行号等等

```elixir
defmacro __ENV__()
```

这个宏包含了当前环境信息。你可以从中拿到当前的文件名/行号/函数名等等

```elixir
iex(4)> __ENV__.file
"iex"

iex(5)> __ENV__.line
5
```

## 8. 手动创建 Pids

在Elixir中你可以用`pid` 函数手动创建 pid ，函数支持两种方式。

### def pid\(string\)

用字符串创建 **PID** ：

```elixir
iex> pid("0.21.32")
#PID<0.21.32>
```

### def pid\(a, b, c\)

通过三个正整数来创建 **PID** ：

```elixir
iex> pid(0, 21, 32)
#PID<0.21.32>
```

### 为什么需要手动去创建 pids ？

假设你正在开发一个库的一个函数，该函数的需要一个 pid 的入参，你想测试函数是，就需要手动创建一个 pid 。

你不能通过这样 `pid = #PID<0.21.32>` 来创建，因为 `#` 是注释符。

```elixir
iex(6)> pid = #PID<0.21.32>
...(6)>
```

像上面那样， **iex** 还在等待更多输入，因为 `#PID<0.21.32>` 已经被当成了注释。

所以你只能通过其他数据来完结这个表达式。你可以试试看：

```elixir
iex(6)> pid = #PID<0.21.32>      # 表达式未完成
...(6)> 23    # 输入 23
23            # 表达式已完成
iex(7)> pid
23
```

## 9. 字符串替换时的 global 参数

`String.replace` 方法可以将某些字符串替换为其他字符串。默认情况下，它会替换掉所有出现的字符串。看下面的示例：

```elixir
iex(1)> str = "hello@hi.com, blackode@medium.com"    
"hello@hi.com, blackode@medium.com"

iex(2)> String.replace str,"@","#"
"hello#hi.com, blackode#medium.com
```

`String.replace str, "@", "#" 相当于 `String.replace str, "@", "#", global: true`

但是，当你只需要替换第一个字符串时，你可以通过指定 `global: false` 参数来实现。所以下面例子中的 `@` 只会被替换一次：

```elixir
iex(3)> String.replace str, "@", "#", global: false
"hello#hi.com, blackode@medium.com"
```

只有一个 `@` 被替换为 `#`。

## 10. 内存使用情况

你可以通过 `:erlang.memory` 获取当前内存使用(单位 bytes)情况。

```elixir
iex(1)> :erlang.memory
[total: 16221568, processes: 4366128, processes_used: 4364992, system: 11855440,
 atom: 264529, atom_used: 250685, binary: 151192, code: 5845369, ets: 331768]
```

另外，你还可以通过给定参数来访问 `:erlang.memory :atom`  获取 atoms 的内存使用情况。

```elixir
iex(2)> :erlang.memory :atom
264529
```

[前一篇](part3.md) [下一篇](part5.md)
