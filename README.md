# Fx
A small embeddable scripting language and virtual machine written in C

Fx is a small embeddable scripting language and virtual machine written in C.
This document describes the language and runtime available in **Fx v0.1**.

Fx has a Lua-inspired runtime, a compact bytecode VM, optional type annotations,
objects, pattern-style matching, and a pipeline operator.

## Status

Fx v0.1 is an experimental language. The VM and syntax are usable for small
scripts, but the language is still evolving.

Implemented in this version:

- Numbers, strings, booleans, nil, objects, and functions.
- Local and global variables.
- Functions, closures, returns, and varargs.
- Optional type annotations.
- Object literals and indexed/member access.
- `if`, `while`, `repeat`, numeric `for`, generic `for`, and `switch`.
- `match` expressions.
- The `|>` pipeline operator.
- Lua-like `print` and standard libraries for I/O, strings, math, objects, and OS operations.

Known limitations:

- Type information for arbitrary object members and inherited members is not yet
  checked as deeply as local variables and function parameters.
- The language does not currently provide modules, a package manager, or a
  separate compiler executable.

## Building

### Windows with GCC

From the directory containing `fx.c`:

```powershell
gcc -std=c99 -O2 fx.c -lm -o fx.exe
```

Run a script with:

```powershell
.\fx.exe hello.fxls
```

### Windows with Tiny C Compiler (tcc)

If you prefer a lightweight compiler, install **tcc** (Tiny C Compiler) from
<https://bellard.org/tcc/> or via a package manager. Compile the source with:

```powershell
tcc -std=c99 -O2 fx.c -lm -o fx.exe
```

The resulting `fx.exe` can be run the same way:

```powershell
.\fx.exe hello.fxls
```

### Linux or macOS

```sh
gcc -std=c99 -O2 fx.c -lm -o fx
./fx hello.fxls
```

`-lm` links the C math library used by the `math` module.

### Linux (Ubuntu)

On Ubuntu, install the build tools and compile with GCC:

```sh
sudo apt-get update
sudo apt-get install build-essential
gcc -std=c99 -O2 fx.c -lm -o fx
./fx hello.fxls
```

If you would rather use **tcc** on Ubuntu:

```sh
sudo apt-get install tcc
tcc -std=c99 -O2 fx.c -lm -o fx
./fx hello.fxls
```

The program accepts the script path as its first argument. Additional command
line arguments are made available through the global `arg` object.

If no script path is supplied, the executable exits after initializing the VM.

## A First Program

```fxls
print("Hello from Fx")

x: number = 1
x |> math.cos |> print
```

Output is similar to:

```text
Hello from Fx
0.54030230586814
```

## Lexical Rules

### Comments

A semicolon starts a comment that continues to the end of the line:

```fxls
; this is a comment
print("hello") ; trailing comment
```

### Strings

Single-quoted and double-quoted strings are supported:

```fxls
name = "Fx"
message = 'hello'
```

Common escape sequences include `\n`, `\r`, `\t`, `\a`, `\b`, `\f`, and `\v`.

### Statements and separators

Statements are normally separated by newlines. A semicolon can also separate
statements, although semicolons are additionally used for comments when they
appear at the start of a line or after code.

Blocks are closed with `end`.

## Values and Types

Fx v0.1 has these built-in type annotations:

| Annotation | Meaning |
| --- | --- |
| `any` | Any value; the default when no annotation is present |
| `number` | A floating-point number |
| `string` | A string |
| `boolean` | `true` or `false` |
| `nil` | The nil value |
| `object` | An object/table value |
| `function` | A function value |

Examples:

```fxls
count: number = 10
name: string = "Ada"
enabled: boolean = true
nothing: nil = nil
config: object = {}
```

The compiler checks explicit assignments when it can determine both the
expected and actual types. `any` and unknown expression types are accepted.

Boolean values:

```fxls
ready = true
missing = false
empty = nil
```

Numbers are represented as floating-point values and support decimal literals.

## Variables and Assignment

Global assignment:

```fxls
x = 10
x = x + 2
```

Local variables:

```fxls
local total: number = 0
local label = "total"
```

Multiple assignment and declarations are supported by the parser:

```fxls
local first, second = 1, 2
```

Member and indexed assignment:

```fxls
person = { name = "Ada", age = 36 }
person.age = 37
person["name"] = "Grace"
```

## Operators

### Arithmetic

```text
+   addition
-   subtraction and unary minus
*   multiplication
/   division
%   remainder
^   exponentiation
```

### Comparison and logic

```text
==  equal
~=  not equal
<   less than
<=  less than or equal
>   greater than
>=  greater than or equal
and logical and
or  logical or
not logical negation
```

### Other operators

```text
..   string concatenation
#    length
|>   pipeline
```

Concatenation example:

```fxls
x = "Point " .. 4 .. ", " .. 3
print(x)
```

### Pipeline operator

`a |> b` evaluates `a` and passes it as the first argument to `b`.
The right-hand side is normally a function or a member expression that
resolves to a function.

```fxls
1 |> print
1 |> math.cos |> math.sin |> math.tan |> print
```

This is equivalent in intent to:

```fxls
print(1)
print(math.tan(math.sin(math.cos(1))))
```

The pipeline is especially useful for readable data transformations.

## Functions

The keyword `fun` declares a named function:

```fxls
fun add(a: number, b: number): number
    return a + b
end

print(add(2, 3))
```

Functions can also be anonymous:

```fxls
local twice = fun(value: number): number
    return value * 2
end

print(twice(5))
```

Parameter and return annotations are optional:

```fxls
fun greet(name)
    return "Hello, " .. name
end
```

A function may use varargs:

```fxls
fun show(...)
    print(...)
end
```

Member-call syntax passes the receiver as `self`:

```fxls
object = { value = 4 }
object:method()
```

## Control Flow

### If

```fxls
if score >= 10 then
    print("high")
elif score >= 5 then
    print("medium")
else
    print("low")
end
```

### While

```fxls
local i = 0
while i < 3 do
    print(i)
    i = i + 1
end
```

### Repeat

```fxls
local value = 0
repeat
    value = value + 1
until value >= 3
```

### Numeric for

```fxls
for i = 1, 5, 1 do
    print(i)
end
```

The step is optional and defaults to `1`.

### Generic for

The VM also supports the Lua-style iterator form:

```fxls
for key, value in pairs(object) do
    print(key, value)
end
```

### Break

```fxls
while true do
    break
end
```

## Objects

Object literals use braces:

```fxls
user = {
    name = "Ada",
    age = 36
}

print(user.name)
print(user["age"])
```

Objects can contain positional values as well as named fields:

```fxls
values = { 10, 20, 30 }
print(values[1])
```

## `switch`

`switch` evaluates a subject and provides `case` and `default` blocks:

```fxls
switch value
    case 1
        print("one")
    end
    case 2
        print("two")
    end
    default
        print("other")
    end
end
```

The current v0.1 implementation uses the blocks for control flow but does not
provide a separate `break` requirement between cases.

## `match` Expressions

`match` is an expression. Each pattern is followed by `:` and its result;
`::` introduces the default result:

```fxls
label = match value
    1: "one"
    2: "two"
    :: "other"
end

print(label)
```

The selected branch value becomes the value of the whole expression.

## Standard Library

The VM opens these libraries automatically:

- Base globals
- `object`
- `io`
- `os`
- `string`
- `math`

### Base functions

Available global functions include:

| Function | Purpose |
| --- | --- |
| `assert(value, message)` | Raise an error when a value is false |
| `error(message)` | Raise a runtime error |
| `import(name)` | Load and execute a `.fxls` module |
| `require(name)` | Alias of `import` |
| `loadfile(path)` | Load a file as a function |
| `loadstring(source)` | Compile source text as a function |
| `pcall(function, ...)` | Protected call returning success and results |
| `rawget(object, key)` | Read a field without metamethod lookup |
| `setmetaobject(object, meta)` | Set an object's metatable |
| `tonumber(value, base)` | Convert a value to a number |
| `tostring(value)` | Convert a value to text |
| `type(value)` | Return the runtime type name |
| `unpack(object)` | Return object elements as multiple results |
| `next(object, key)` | Iterate object fields |
| `pairs(object)` | Create a key/value iterator |
| `ipairs(object)` | Create an integer-index iterator |
| `print(...)` | Print values separated by tabs and followed by a newline |

### `io`

The I/O library includes:

```fxls
io.write("hello", " ", "world")
io.flush()
```

Available operations include `io.write`, `io.flush`, `io.read`, `io.lines`,
`io.open`, `io.close`, `io.input`, `io.output`, and `io.type`.

The standard output and input handles are available as `io.stdout` and
`io.stdin` when initialized by the runtime.

### `string`

Available functions include:

```fxls
string.byte(text)
string.char(65)
string.find(text, pattern)
string.format("value=%s", value)
string.gmatch(text, pattern)
string.gsub(text, pattern, replacement)
string.lower(text)
string.match(text, pattern)
string.rep(text, count)
string.sub(text, start, finish)
string.upper(text)
```

String methods can also be accessed through the object/metatable behavior when
available.

### `math`

Fx v0.1 currently exposes:

```fxls
math.cos(value)
math.sin(value)
math.tan(value)
```

The functions use radians and return numbers.

### `os`

The OS library is available through `os`. Its exact operations follow the
runtime implementation and the host C library; use `os` functions only where
portability is not critical.

## Modules

A module is a `.fxls` file loaded by name:

```fxls
import("math_helpers")
```

The runtime resolves the module as `math_helpers.fxls` when no extension is
provided. An explicit `.fxls` name is also accepted:

```fxls
require("math_helpers.fxls")
```

Modules execute in the current VM environment. v0.1 does not include a module
cache or package search path.

## Runtime Errors

Errors are reported on standard error with the source name and line when the
parser can provide them:

```text
Error: hello.fxls:3: unexpected symbol
```

Typical causes include:

- Missing `end` for a block.
- Missing `then` after an `if` condition.
- Calling a value that is not a function.
- Indexing a value that is not an object.
- Passing a value with an incompatible explicit type annotation.
- Supplying the wrong number or type of arguments to a native function.

## Embedding

`fx.c` contains the VM, parser, standard libraries, and a command-line entry
point in one translation unit. The public-facing runtime follows a Lua-like C
API shape, including state creation, loading, protected calls, stack access,
objects, strings, and C functions.

For a custom host application, reuse the VM initialization and library-opening
functions from `fx.c`, then load source with the buffer/file loading functions
and execute it through a protected call. The current executable's `main`
function is the simplest reference host.

## Origin and Attribution

The Fx VM and parser started from `minilua.c`, the small self-contained Lua
host/compiler used by LuaJIT. Fx keeps the general shape of that implementation
while adapting the lexer, parser, bytecode layer, runtime objects,
type annotations, standard libraries, and Fx-specific syntax.

Upstream reference:

- [LuaJIT `minilua.c`](https://github.com/LuaJIT/LuaJIT/blob/master/src/host/minilua.c)
- [LuaJIT repository](https://github.com/LuaJIT/LuaJIT)

See [NOTICE](NOTICE) for the Fx project notice and
[THIRD_PARTY_NOTICE](THIRD_PARTY_NOTICE) for the LuaJIT attribution and
upstream licensing information.

## Project Layout

```text
fx.c       VM, parser, runtime, standard libraries, and CLI
hello.fxls  Fx source script
README.md  This documentation
```

## License

See [LICENSE](LICENSE) for the licensing terms.

Project notices are maintained separately in [NOTICE](NOTICE) and
[THIRD_PARTY_NOTICE](THIRD_PARTY_NOTICE).
