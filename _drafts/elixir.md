# Elixir

Elixir is an open-source functional programming language for the Erlang virtual machine.

In turn, Erlang is an open-source software development platform consisting of:

- the Erlang programming language, which is a simple functional language with basic concurrency primitives
- the Erlang virtual machine, called BEAM, which has the main task of scheduling the execution of so-called Erlang processes, which are very lightweight units of execution that can be run concurrently, in numbers of thousands or even millions
- the OTP framework, providing abstractions for concurrency and distribution patterns, error detection and recovery, packaging, deployment and live updates
- various tools taking care of compiling the language into the BEAM bytecode, starting the BEAM instance, creating deployable releases, running the interactive shell, etc.

Erlang allows to develop highly available systems through the following properties:

- processes are completely isolated from each other, by sharing no memory and running independently, so that a crash of a process doesn't affect the others
- processes communicate with each other only through asynchronous messaging, making it much easier to work concurrently by not requiring any synchronization mechanism like locks, mutexes and semaphores
- the Erlang scheduler is preemptive, meaning that it gives a small execution window to each process, and then pauses it and runs another process
- I/O operations are internally delegated to separate operating system threads, or to a kernel-poll service
- garbage collection is performed independently per process

Thanks to this design, Erlang achieves the following properties, which ultimately provide high availability:

- *fault-tolerance*: Erlang processes allow to strictly localize unexpected situations like errors, bugs and crashes, which only affect a small part of the system, and can often be tackled by simply starting a new process in place of the failed one
- *scalability*: since processes are isolated, and communication is always asynchronous, they can be vertically scaled, meaning updated in software or upgraded in infrastructure, without requiring a whole system restart
- *distribution*: since processes are independent they can be distributed over multiple CPUs, possibly on different nodes of a network, thus enabling horizontal scalability
- *responsiveness*: since processes are executed preemptively, I/O operations are run in parallel, and garbage collection is per-process, no long-running process can halt the entire system, and thus the overall responsiveness of the system is not degraded

While Erlang is designed to build highly available systems, Elixir provides a more user-friendly and productive experience to programmers, by bringing in features from other languages such as Clojure and Ruby, and more modern development tools for testing, packaging, etc. Elixir is a language that targets the Erlang virtual machine, meaning that the result of the compilation of Elixir source files is valid BEAM bytecode, that can be run on a BEAM instance: this means that Elixir is usually as performant as Erlang, and can seamlessly use existing Erlang libraries.

## Modules

Modules are the broader form of organization of code in Elixir. Any code in Elixir must be written inside modules.

To define a new module we use the `defmodule` macro:

```elixir
defmodule Geometry do
  ...
end
```

The name of the module must start with an uppercase letter, and contain alphanumeric characters, underscores, or dots. The community convention for module names is pascal case.

Modules can be defined as nested one inside another:

```elixir
defmodule Geometry do
  defmodule Rectangle do
    ...
  end
end
```

This is actually syntactic sugar that Elixir allow us to use, but which ultimately results in two different modules being created: `Geometry` and `Geometry.Rectangle`. In fact, dots are conventionally used to define hierarchies of modules

However, using dots inside a module name has no real effect in the way Elixir treats that module, because in the final BEAM bytecode all modules will be defined at the same level, with no hierarchy whatsoever.

## Functions

Functions are the main way we use in Elixir to organize our code. Functions must always be defined inside a module, and to do this we use the `def` macro:

```elixir
defmodule Geometry do
  def rectangle_area(a, b) do
    a * b
  end
end
```

Function names must start with a lowercase alphabetic character, or an underscore, can contain any number of alphanumeric characters or underscores, and can end with a question mark `?`, conventionally stating that the function is evaluated to a boolean value `true` or `false`, or with an exclamation mark `!`, stating that the function might produce a runtime error. The community standard for function naming is snake-case.

The return value of a function is the value of the last expression found in the body, which in this example is `a * b`.

If the function body only contains one statement, we can use the following simplified form:

```elixir
def rectangle_area(a, b), do: a * b
```

If the function takes no argument, we can omit the argument list in the definition:

```elixir
def run do
  ...
end
```

To call a function we need to identify the function we want to call by its name, and then specify its arguments as a comma-separated list of values between parenthesis:

```elixir
function_name(args)
```

Parenthesis are actually optional, so we can omit them:

```elixir
rectangle_area 3, 2
```

When calling a function from within the same module where it's been defined, we can avoid to specify its module:

```elixir
defmodule Geometry do
  def rectangle_area(a, b), do: a * b
  def square_area(a), do: rectangle_area(a, a)
end
```

here we are calling `rectangle_area` from within `square_area` because they're both defined inside the `Geometry` module.

Function arity is the number of arguments taken by a function. Thus, a function is uniquely identified by its containing module, its name, and its arity. For example, the following function:

```elixir
defmodule Rectangle do
  def area(a, b) do
    ...
  end
end
```

can be referred to by `Rectangle.area/2`, where the `/2` part is the function's arity, meaning it takes two arguments. It's important that the arity can be added to the function identifier because we can define different functions with the same name, but different number of arguments:

```elixir
defmodule Rectangle do
  def area(a), do: area(a, a)
  def area(a, b), do: a * b
end
```

here the first function can be identified with `Rectangle.area/1`, while the second with `Rectangle.area/2`.

Different variations of the same function can be automatically generated by defining default arguments:

```elixir
defmodule MyModule do
  def fun(a, b \\ 1, c, d \\ 2) do
    a + b + c + d
  end
end
```

here we are stating that the argument `b` has the default value of `1`, and the argument `d` has the default value of `2`. This in practice has the effect that three distinct functions will be created: `MyModule.fun/2` taking values only for `a` and `c`, `MyModule.fun/3` taking values only for `a`, `c` and `d`, and `MyModule.fun/4`, taking values for all four arguments. Additionally, for this reason it's not possible in Elixir to define functions with a variable number of arguments.

## Statements

An Elixir program is ultimately a collection of statements, each of which instructs the BEAM virtual machine to perform some task. For example the following statement:

```elixir
IO.puts("Hello, world!")
```

is a function call that has the effect of printing a string to the standard output.

A statement is a line, or a collection of lines, of source code, meaning that a statement is terminated just by a newline character.

Every statement in Elixir is also an expression, meaning that it can be evaluated, and used as a value. For example:

```elixir
IO.puts(IO.puts("Hello, world!"))
```

will first print `"Hello, world!"`, and then print the return value of the inner call, representing the successful execution of the print operation.

## Variables

Elixir is a dynamic programming language, and thus variables don't have a type attached to them. The type of a variable is instead determined by the actual value is bound to it at a certain time. To bind a value to a variable we use the binding statement:

```elixir
monthly_salary = 10000
```

which, as an expression, can be evaluated to the value that has been assigned.

Once a variable has been bound, it can be used as an expression, producing the value that has been bound to it.

```elixir
monthly_salary # evaluated to 10000
```

Variable names follow the same convention as function names.

Variables can be rebound to different values. What this means is that new memory is allocated to store the new value, and the variable is set to point to the new memory location; the original value is kept in memory until released by the garbage collector. This actually means that no memory location is ever overwritten (before it's freed), and thus that values themselves are actually immutable. Memory becomes eligible for garbage collection when the variable bound to it goes out of scope.

## Pipelines

Pipelines are syntactic sugar to combine multiple function calls where the output of a function is passed as input to the next:

```elixir
-5 |> abs() |> Integer.to_string() |> IO.puts()
```

this means that the initial value `-5` is first run through the `abs()` function call, producing `5`, which is then run through the `Integer.to_string()` function call, producing `"5"`, which is lastly printed to the screen with `IO.puts()`. The pipeline syntax is actually a macro that produces a series of nested calls after compilation, which in this case would be:

```elixir
IO.puts(Integer.to_string(abs(-5)))
```

When functions take more than one parameter, the pipeline operator pipes the result of the previous call to the first argument of the next, so that:

```elixir
prev(arg1, arg2) |> next(arg3, arg4)
```

is equivalent to:

```elixir
next(prev(arg1, arg2), arg3, arg4)
```

## Tools

The `mix` tool provides a way to automatically format code according to the community standard for coding style:

```shell
$ mix format
```

## References

"Elixir in Action, Second Edition" by Sasa Juric, Manning 2019
