<h1 align="center">CText</h1>

<p align="center">CText is a tool for processing text.</p>

## Installation

Use the following command to install CText. It requires Python >3.11 to work.

```bash
curl -fsSLO "https://raw.githubusercontent.com/arielherself/ctext/main/ctext" && chmod +x ctext && sudo mv ctext /usr/local/bin
```

## Configuration

To start using CText, you need to create a configuration file `CTextLists.toml` which is composed of some *functions*. Here is an example configuration:

```toml
[debug]
nargs = 0
comma = false
permissive = true
value = 'std::cerr $debug_expand(#...#) << std::endl;'

[debug_expand]
nargs = 0
comma = false
permissive = true
value = '$arrow_out(#0#) $debug_expand(#1...#)'

[arrow_out]
nargs = 1
comma = false
permissive = false
value = '<< "#0# = " << (#0#) << ", "'
```

If we use it in a C++ file like this:

```cpp
$debug(x,y,z)
```

then the following code will be generated by CText:

```cpp
std::cerr << "x = " << (x) << ", " << "y = " << (y) << ", " << "z = " << (z) << ", "  << std::endl;
```

### `nargs`

Number of arguments that this function receives. It's only for verification in strict mode.

### `comma`

The **comma-bypass** feature allows you to forward the argument including a comma (`,`) received from caller to another function. For example:

```toml
[define]
nargs = 2
comma = true
permissive = false
value = '$__replace_with(#0#,#1#)'
```

If we define this function called `define` like this, then we must ensure that the comma in the argument won't be interpreted as a separator in the argument list during the function expansion.
Suppose we have `comma = false` and this function call:

```cpp
$define(pair_gen,x\,y)

int x, y; std::cin >> x >> y;
std::pair<int, int> my_pair = {pair_gen};
```

We expect `pair_gen` to expand to `x,y`. But instead it's `x` because the `define` function expands to `$__replace_with(pair_gen,x,y)` and the built-in function `__replace_with` replaces all occurrences of the first argument with the second argument.

On the other hand, if we have `comma = true` set then `define` will expand to `$__replace_with(pair_gen,x\,y)` as the comma-bypass feature treats the comma in an argument as an escape character and keeps the backslash all the time.

### `permissive`

A permissive function gracefully expands to nothing when there is an index overflow in the argument list, like when it tries to get the 3rd argument `#2#` although there are only no more than 2 arguments passed to it.
This feature makes it easier to write recursive functions. Take the first configuration above as an example.
In function `debug_expand` We use an argument range `#1...#` to call itself recursively, and when there is no `#0#` during a call, the function automatically finish the recursion.

If a function is **not** permissive, it will ignore any occurrence of overflowed parameters, placing nothing on its position in the expanded expression.
