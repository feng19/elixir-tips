<img class="elixirtip-img" src="assets/images/parts/elixir_tips_5.jpg"/>

### 1. 查看 Mix 默认的编译器列表

```elixir
iex> Mix.compilers
[:yecc, :leex, :erlang, :elixir, :xref, :app]
```

另外你也可以通过修改 `mix.exs` 来新增编译器，加在前面或者后面都可以：

```elixir
#mix.exs
def project do
 [compilers: Mix.compilers ++ [:gettext]
end
```

### 2. 选取列表中的元素

众所周知，一个列表由 `head` 和 `tail` 组成，像这样： `[head | tail]`。利用此原理，我们可以像下面这样去获取列表中的元素：

```elixir
iex> [first | [second | [third | [ fourth | _rest ]]]] = [1, 2, 3, 4, 5, 6, 7]
[1, 2, 3, 4, 5, 6, 7]
iex> first
1
iex> {second, third, fourth}
{2, 3, 4}
```

其实我们可以用下面这种简单的方式来达到跟上面一样的效果：

```elixir
iex> [first, second, third, fourth | _rest] = [1, 2, 3, 4, 5, 6, 7]
[1, 2, 3, 4, 5, 6, 7]
iex> first
1
iex> {second, third, fourth}
{2, 3, 4}
```

### 3. get\_in /Access.all\(\)

我们都了解 `get_in` 能获取一个多层嵌套 map 里对应 key 的值：

```elixir
iex> user = %{"name" => {"first_name" => "blackode", "last_name" => "john" }}
%{"name" => %{"first_name" => "blackode", "last_name" => "john"}}
iex > get_in user, ["name", "first_name"]
"blackode"
```

但是，如果我们有一个 map 列表，如：`[maps]` ，需要提取每个 map 的 `first_name` 的值，通常我们会使用 `enum` 来实现。其实我们也可以结合 `get_in` 和 `Access.all()` 来实现相同的效果：

```elixir
iex> users=[%{"user" => %{"first_name" => "john", "age" => 23}},
            %{"user" => %{"first_name" => "hari", "age" => 22}},
            %{"user" => %{"first_name" => "mahesh", "age" => 21}}]
# 一个 map 列表
iex> get_in users, [Access.all(), "user", "age"]
[23, 22, 21]
iex> get_in users, [Access.all(), "user", "first_name"]
["john", "hari", "mahesh"]
```

**注意:** 如果 key 不存在于 map，会返回 `nil`：

```elixir
iex(17)> list = [%{name: "john"}, %{name: "mary"}, %{age: 34}]
[%{name: "john"}, %{name: "mary"}, %{age: 34}]
iex(18)> get_in(list, [Access.all(), :name])
["john", "mary", nil]
```

**警告:** 当你将 `get_in` 结合 `Access.all()` 一起使用时，像上面那样，第一个的参数的数据类型必须是一个列表，其位置顺序必须与 `Access.all()` 的位置能对应上，如果是其他类型，将会报错：

```elixir
iex(19)> get_in(%{name: "blackode"}, [Access.all(), :name])
** (RuntimeError) Access.all/0 expected a list, got: %{name: "blackode"}
    (elixir) lib/access.ex:567: Access.all/3
```

另外，你可以改变 Access.all\(\) 在列表中的位置。但是该位置的前一个 key 对应的值必须是列表。往下进一步了解更多。

#### 深入了解

同样地，我们也可以将 `Access.all()`  用在 `update_in`，`get_and_update_in`，等等类似的函数中： 
例如，给 user(map) 的 books(list) 字段放置一个列表的书，怎么将这些列表内的 书(map) 的名字都转成大写：

```elixir
iex> user = %{name: "john", books: [%{name: "my soul", type: "tragedy"}, %{name: "my heart", type: "romantic"}, %{name: "my enemy", type: "horror"}]}
iex> update_in user, [:books, Access.all(), :name], &String.upcase/1
%{books: [%{name: "MY SOUL", type: "tragedy"}, %{name: "MY HEART", type: "romantic"}, %{name: "MY ENEMY", type: "horror"}], name: "john"}
iex> get_in user, [:books, Access.all(), :name]
["my soul", "my heart", "my enemy"]
```

首先，与上一个例子不同的是 users 是一个列表，这个例子中 user 的类型并不是一个列表。但是，我们改变 Access.all\(\) 在列表中的位置：`[:books, Access.all(), :name]`，`:books` 字段对应的值返回的是一个列表类型，如果是其它类型的话，将会报错。

### 4. 数据推导与过滤器一起用

我们实现数据推导通过这种形式： `for x <- [1, 2, 3], do: x + 1` 。但我们也可以让它与过滤器一起用。

#### 常规用法

```elixir
iex> for x <- [1, 2, 3, 4], do: x + 1
[2, 3, 4, 5] 
# 这是我们常规的用法，但是现在我们应该跳出框框思考一下
```

#### 与过滤器一起用

这里我们有两个整数列表，将他们交叉相乘，并过滤掉奇数的乘积：

```elixir
iex> for x <- [1, 2, 3, 4], y <- [5, 6, 7, 8], rem(x * y, 2) == 0, do: {x, y, x * y}
[{1, 5, 5}, {1, 7, 7}, {3, 5, 15}, {3, 7, 21}]
# 这里的 rem(x * y, 2) 实际作用为过滤器
```

## 5. Binary/字符串 推导

Binary/字符串 推导区别不大，只需要用  `<<>>` 包裹一下：

```elixir
iex> b_string = <<"blackode">>
"blackode"
iex> for << x <- b_string >>, do: x + 1
'cmbdlpef'
# 这里的打印结果里面的字母，只是将 "blackode" 的每个字母换成其字母表后面一个的字母
```

细心观察你会发现：原来的列表推导形式 `x <- b_string` ，变成了这样： `<< x <- b_string >>` 。

## 6. IO.stream 推导优势

我们推高一个级别来继续谈谈推导。
我们从键盘读取输入，然后将内容转换成大写字母，然后继续等待下一个输入：

```elixir
for x <- IO.stream(:stdio, :line), into: IO.stream(:stdio, :line), do: String.upcase(x)
```

基本上来说 `IO.stream(:stdio, :line)` 会从键盘读取输入，另外也支持打印到控制台，推导中的 `into`选项是将装换后的内容打印出来：

```elixir
iex> for x <- IO.stream(:stdio, :line), into: IO.stream(:stdio, :line), do: String.upcase(x)
hello
HELLO
hi
HI
who are you?
WHO ARE YOU?
blackode
BLACKODE
^c ^c # to break
```

## 7. 用一行定义多个别名

```elixir
alias Hello.{One,Two,Three}
#The above line is same as the following 
alias Hello.One
alias Hello.Two
alias Hello.Three
```

## 8. 导入下划线前缀的函数

默认情况下，前缀为 \_ 不会被导入。但是，通过设置参数 `:only` 指定导入。

```elixir
import File.Stream, only: [__build__: 3]
```

## 9. 子字符串

Elixir中没有像 `sub_str` 的函数，但是你可以通过 `String.slice/2` 函数来实现：

```elixir
iex> String.slice("blackode", 1..-1)
"lackode"
iex> String.slice("blackode", 0..-4)
"black"
```

## 10. 字符串拼接

字符串拼接有两种方式。

```elixir
iex> str1 = "hello"
iex> str2 = "blackode"
```

以上面的代码作为实例……

#### 字符串插值

```elixir
iex> mystring = "#{str1}#{str2}"
helloblackode
```

#### 使用 &lt;&gt; 操作符

```elixir
iex> mystring = str1 <> str2
helloblackode
```

这种拼接方式是最好的，推荐使用。

如果你想拼接字符串列表： `["hello", "blackode"]` ，可以使用 `Enum.join`

```elixir
iex> mystrings = ["hello", "blackode"]
["hello", "blackode"]
iex> Enum.join(mystrings)
"helloblackode"
```

[前一篇](part4.md) [下一篇](part6.md)
