# C++ Crash Course

## Statements, values and expressions

A C++ program is defined as source code written in text files, for example:

```c++
#include <cstdio>

int main() {
  printf("Hello, world!");
  return 0;
}
```

This source file then has to be built into an executable application, that can be run on the operating system.

The ultimate purpose of a program is to manipulate a given set of data, the *input*, to produce another, meaningful, set of data, the *output*. In order to do so, any C++ program is defined as a sequence of *statements*, which are instructions telling the program to perform specific operations with the given data.

For example, in the previous program we are using three different statements:

- `int main() {` defines a special section of code, that every C++ program must contain, representing the point where the execution should to start
- `printf("Hello, world!");` causes the message "Hello, world!" to be printed to the standard output of the executing process
- `return 0;` tells the executing process to return the exit status `0` to the operating system upon termination

The line `#include <cstdio>` is not a statement, but a preprocessor directive: this is a building feature that is used to simplify the process of writing source files, and its effect is that at build time it will be replaced with the contents of another file, belonging to the `stdio` external library, containing other statements. The end result is that the actual source file will eventually contain only actual statements. In this particular case, including this `stdio` file is necessary because it contains the explanation of what that `printf` written below means.

This example program, in particular, has no input data, but produces, as output data, the message "Hello, world!", and the exit status `0`.

The data a program works with is made of *values*, which are meaningful units of data, like the numbers `1`, `2` and `3`, or the message `"Hello, world!"`.

The easiest way to use a value in a statement is with its *literal notation*, which is a way to represent that value by using characters that can be included directly in the source code:

```c++
printf("%d", 1);
```

where we are referring the two values `"%d"` and `1` in literal notation.

An *expression* is an operation performed on values, that produces another value as a result. Calculating the result of an expression is called *evaluating* the expression.

A single value, like `1`, is itself an expression, which is evaluated to that same value. The operations that can be performed on values can be written with infix notation, or with call notation.

There is only a limited set of operations, defined in the C++ language, that can be applied with *infix notation*, and they are called *operators*. Operators are generally written between their operands ("infix"), like the `+` operator, which performs a sum of two operands, and is used like in `1 + 2`. There are, however, operands that work differently, like the `!` operator, which performs a logical negation on a single operand, and is written before its operand, like in `!true`, or the operator `?:`, which is applied to three operands, and performs a conditional choice, like in `1 == 2 ? 'a' : 'b'`.

All other operations are performed with *call notation*, where the name of the operation is followed by the list of operands, included between parenthesis, like in `printf("%d", 1)`. Unlike operators, these operations are defined within the program, and thus there can be any number of them.

We can use expressions wherever a value is expected, and in that case the expression is evaluated first, and then its value is used in place of the expression itself:

```c++
printf("%d", 1 + 2); // prints "3"
```

Since expressions can be used in place of values, we can produce indefinitely complex expressions by using expressions inside other expressions:

```c++
printf("%d", 1 + 2 - 3); // prints "0"
```

here the expression `1 + 2 - 3` is actually the sum of the value `1` and the other expression `2 - 3`: in order to evaluate this expression, we first need to evaluate the inner expression `2 - 3`, which evaluates to `-1`, so that the first expression is actually `1 - 1`, which evaluates to `0`.

## Types, variables and the assignment statement

Values can be grouped into *types* according to the operations that can be performed on them: for example the values `1`, `2`, `3`, etc., are of type `int`, because they all support the typical operations defined for integral numbers, like addition, multiplication, etc.

At a more fundamental level, types are related to the size in memory that is needed to represent their values: in fact in order to perform an operation on a value, the implementation of that operation needs to know what's the layout of bytes that value is represented in memory with, and thus all values sharing the same memory layout can also be used with the same operations.

Some types are already provided by the C++ language out of the box, while the others are custom-defined. Some types are called *primitive types*, because they are the most simple ones, while the others are called *composite types*, because they are defined as aggregates of primitive types. All primitive types, and some composite types are provided out of the box by C++, while all custom types are composite types. Primitive types are also the only ones that can be represented in the code with a literal notation.

Values can be assigned meaningful names to, in order to better understand their meaning, and to refer to results of computations that might be decided only when the program is run, for example if they depend on some input data. A name for a value is called a *variable*, and it's created with a *variable declaration statement*, where we specify the type of the variable, its name, and its initialization list:

```c++
int result { 1 };
```

Once we have a variable that stands for some value, we can use it in place of the value itself:

```c++
printf("%d", result); // prints "1"
```

meaning that a variable is itself an expression, a *variable expression*, which is of course evaluated to the value assigned to it.

Variables are called as such because the value they refer to can be changed during the course of the execution. To give to a variable a different value we use the *assignment operator* `=` in an *assignment statement*:

```c++
result = 2;
```

from now on the `result` variable will refer to the value `2`:

```c++
printf("%d", result); // prints "2"
```

The assignment operator takes two operands: the first is the one that will be assigned a value, and the second is the value that will be assigned to the first. In order for this to be possible, the expression to the left of the assignment statement has to produce a so-called *lvalue*, which is a value to which some other value can be assigned to, and the expression to the right has to produce a so-called *rvalue*, which is a value that can be assigned to another value. Only some special values are lvalues, and the most common of them are variables. On the other hand, it's much easier to produce rvalues, because literally every possible value is an rvalue.

Assignment statements happen to also be expressions, meaning that they can be evaluated, and their value is exactly the value that is being assigned to the variable:

```c++
int result { 1 };
printf("%d", result = 2); // prints "2"
```

here the second operand of `printf` is the entire assignment expression `result = 2`, which is evaluated to the value `2`, resulting in that same value `2` to eventually be passed to `printf`.

## Primitive types

### Integers

Integer types are used to represent whole numbers, like `1`, `-34` and `3622534`. C++ supports the following sizes of integers:

- `short int`: represents small integers, of 2 bytes in size
- `int`: represents average integers, of 4 bytes in size
- `long int`: represents big integers, of 4 bytes or 8 bytes depending on the architecture and OS for which the program is compiled
- `long long int`: represents very big integers, of 8 bytes

We can omit the `int` part of the type when writing integers types in code, except when it's just `int`.

Integer types support the `signed` and `unsigned` qualifiers: `signed` integers can represent both positive and negative integers, while `unsigned` integers can only support non-negative integers. This means that `unsigned` integers allow to represent twice as many numbers that `signed` integers.

We can omit the `signed` qualifiers, because integers are always assumed to be `signed` except when `unsigned` is explicitly added.

Thus, the following shorthand notations are available to refer to integer types:

- `short int`, `short`: equivalent to `signed short int`
- `unsigned short`: equivalent to `unsigned short int`
- `int`: equivalent to `signed int`
- `signed long`, `long`: equivalent to `signed long int`
- `unsigned long`: equivalent to `unsigned long int`
- `signed long long`, `long long`: equivalent to `signed long long int`
- `unsigned long long`: equivalent to `unsigned long long int`

Integer values can be referred to in code with literal representation, by writing the numbers using number characters as usual, optionally adding a prefix, a suffix, or any number of single quote characters `'`.

The prefix indicates what representation is being used for writing the number:

- `0b` for binary representation, like `0b0010110111101100`
- `0` for octal representation, like `0026754`
- no prefix for decimal representation, like `11756`
- `0x` for hexadecimal representation, like `0x2dec`

The suffix indicates what's the exact integer type of the value. If no suffix is used, the compiler chooses the `int` type if it's big enough to represent the given number, or the smallest `signed` type that fits the given number. Sometimes we want to overwrite this default behavior, for example to let a number be of a type bigger than the smallest one necessary to represent it, or to let a number be `unsigned` (allowing a bigger number to be represented in a smaller space):

- `u` or `U` makes a number `unsigned`, like `11756U`
- `l` or `L` makes a number `long`, like `11756L`
- `ll` or `LL` makes a number `long long`, like `11756LL`

The `u` or `U` prefix can also be combined with one of the other two, like `11756UL` or `11756ULL`.

We can use single quote characters `'` in order to make big integer numbers more readable. There can be any number of single quotes within the number representation, as long as they are located after the first digit (not considering the prefix), and before the last digit (not considering the suffix):

```c++
printf("%d", 0b0010'1101'1110'1100);
printf("%d", 0026'754);
printf("%d", 11'756);
printf("%d", 0x2d'ec);
```

### Floating-point types

Floating point types are used to represent approximations of real numbers, meaning numbers with both an integral and a fractional part, like `3.14` and `-0.33333`. C++ supports the following sizes of floating-point numbers:

- `float`: represents decimal numbers with 4 bytes of precision
- `double`: represents decimal numbers with 8 bytes of precision
- `long double`: represents decimal numbers with 8 bytes of precision

Floating-point values are expressed with a literal notation that must include the dot character `.` as decimal separator, and might include a suffix to specify the type. As with integers, the single quote character `'` can be used to make numbers easier to read. Additionally, floating-point numbers can be written in scientific notation:

- by default a number with a decimal separator is of type `double`, like `3.14`
- in order to force a decimal value to be a `float`, we can add the `F` suffix, like `3.14F`
- in order to force a decimal value to be a `long double`, we can add the `L` suffix, like `3.14L`
- we can write a decimal value with scientific notation, like `0.314e+1`

### The character type

C++ provides a character type to represent single characters that can be used to form textual data. The various character sizes are:

- `char`: stores a character in 1 byte, for example to represent ASCII characters
- `char16_t`: stores a character in 2 bytes, for example to represent UTF-16 characters
- `char32_t`: stores a character in 4 bytes, for example to represent UTF-32 characters
- `wchar_t`: stores a character in a space large enough for the implementation's locale

Character values are represented with literal notation by including a single character in single quotes, like `'c'`.

Character values can be augmented with prefixes to specify the exact type:

- no prefix: `char` characters, like `'c'`
- `u`: `char16_t` characters, like `u'c'`
- `U`: `char32_t` characters, like `U'c'`
- `L`: `wchar_t` characters, like `L'c'`

Some characters cannot be used with the default literal representation because they don't have a visual representation, or could be confused with characters that are used by the C++ language itself. These characters thus need to be represented with an *escape sequence*: the most common ones are the newline character `'\n'`, the single quote `'\''`, and the backslash `'\\'`.

Many unicode characters also have no visual representation, and need to be represented using their Unicode code point, preceded by a prefix:

- `\u`: for 4-digit Unicode points, like `L'\u0041'`
- `\U`: for 8-digit Unicode points, like `L'\U0001F37A'`

### Booleans

C++ provides the only boolean type `bool`, with size of 1 byte. The literal notation for boolean values is simply `true` for true and `false` for false.

### The `std::size_t` type

The `std::size_t` type is used to represent the size of types.

The size of `std::size_t` itself is guaranteed to be enough to represent the maximum size a type can have in the specific implementation, which is usually the size of an `unsigned long long`.

## Basic operators

### Arithmetic operators

Arithmetic operators are applied to numeric values as operands, and produce other numeric values as results.

For integer types:

- `+`: takes two operands, performs an integer sum, like `1 + 2` which evaluates to `3`
- `-`: takes two operands, performs an integer subtraction, like `1 - 2` which evaluates to `-1`
- `*`: takes two operands, performs an integer multiplication, like `1 * 2` which evaluates to `2`
- `/`: takes two operands, performs an integer division (truncating the result), like `1 / 2` which evaluates to `0`

Operands can be of any integer type, and the type of the result will be the smallest type necessary to represent it. For example, in the operation `2147483647 + 1`, both operands are of type `signed int`, but the result would be too big to be represented by another `signed int`, and thus it will be a `signed long int` value.

For floating-point types:

- `+`: takes two operands, performs an approximated decimal sum, like `1.234 + 2.345` which evaluates to `3.579`
- `-`: takes two operands, performs an approximated decimal subtraction, like `1.234 - 2.345` which evaluates to `-1.111`
- `*`: takes two operands, performs an approximated decimal multiplication, like `1.234 * 2.345` which evaluates to `2.89373`
- `/`: takes two operands, performs an approximated decimal division, like `1.234f / 2.345f` which evaluates to `0.526225984096527099609375`

Like integer types, the actual type of the result of a floating-point operation is the smallest type that can represent that result, and this can be bigger than the types of the operands.

### Comparison operators

Comparison operators can be applied to any type, and produce `bool` values as results:

- `==`: takes two operands, tells whether the two operands are equal or not, like `1 == 2` which evaluates to `false`
- `!=`: takes two operands, tells whether the two operands are different or not, like `1 != 2` which evaluates to `true`
- `>`: takes two operands, tells whether the first is greater than the second, like `1 > 2` which evaluates to `false`
- `>=`: takes two operands, tells whether the first is greater than or equal to the second, like `1 >= 2` which evaluates to `false`
- `<`: takes two operands, tells whether the first is less than the second, like `1 < 2` which evaluates to `true`
- `<=`: takes two operands, tells, whether the first is less than or equal to the second, like `1 < 2` which evaluates to `true`

### Logical operators

Logical operators take `bool` operators and produce `bool` values as results:

- `!`: takes one operand, performs a logical negation, like `!true` which evaluates to `false`
- `&&`: takes two operands, performs a logical conjunction, like `true && false` which evaluates to `false`
- `||`: takes two operands, performs a logical disjunction, like `true && false` which evaluates to `true`

### Other operators

The `sizeof` operator takes a single operand, which must be the name of a type, and produces the size of that type, as a `std::size_t` value:

```c++
printf("%zd\n", sizeof(int)); // prints "4"
```

## Arrays

Arrays are one of the composite types provided by C++ out of the box, and represent sequences of values of the same type. The type of an array depends on the type of its elements, and on their number. For example an array of 10 `int` values has type `int[10]`. This means that an `int[10]` array and an `int[20]` array are values of two different types, as much as an `int[10]` array and a `float[10]` array.

For historical reasons, the declaration syntax of a variable of type array does not fully match the usual variable declaration syntax. I particular, instead of writing the whole array type, like `int[4]`, before the variable name, we need to move the square brackets after the variable name:

```c++
int array[4] { 1, 2, 3, 4 };
```

where the initialization list contains all the elements we want to include in our new array.

Since we know from the initialization list how many elements will the array contain, we can avoid repeating this information in the type:

```c++
int array[] { 1, 2, 3, 4 };
```

If we provide the array size in the type declaration, and a single value in the initialization list, an array of the specified size will be created, with all elements initialized to the single value provided:

```c++
int array[4] { 0 };
```

will create an array of 4 `0` values.

An array of `char` can be initialized with a *string literal*:

```c++
char array[] { "Hello" };
```

Once we have an array value containing some elements, we can get back each contained element by using the *index operator* `[]`:

```c++
char array[] { "Hello" };
printf("%c", array[0]); // prints "H"
```

the index operator performs a search by index of an element inside the array, and thus takes the array to search in, and the index of the element to find, as operands, and returns the element of the array that is located at the given index (where the index of the first element is `0`).

The value produced by the index operator applied to an array is actually another example of an lvalue, in addition to variables, meaning that it can appear to the left of an assignment statement, and thus that another value can be assigned to it:

```c++
char array[] { "Hello" };
array[1] = 'a';
printf("%s", array); // prints "Hallo"
```

here we are taking the second element of the array (which has index `1`), and we are replacing it with the new `char` value `a`.

## The conditional statement

The *conditional statement* is used to allow programs to make decisions, meaning to execute a set of statements only if a certain condition applies. A conditional statement is composed by a *condition*, and a *compound statement*. The condition is a boolean expression, which is evaluated to a value of type `bool`; the compound statement is a set of statements grouped between curly braces:

```c++
int x { 0 };
if (x > 0) {
  printf("Positive.");
}
```

here we're using `x > 0` as a boolean expression, which is evaluated to `true` if `x` is strictly greater than `0`, or to `false` otherwise.

If the compound statement contains only one statement, the `if` statement can be also written in a single line, omitting the curly braces, like:

```c++
if (x > 0) printf("Positive.");
```

The `if/else` statement is another form of the conditional statement, where we can also specify a set of statements to be executed in case the condition is *not* met:

```c++
if (x > 0) {
  printf("Positive.");
} else {
  printf("Not positive.");
}
```

Again, in case of single statements we can write this as:

```c++
if (x > 0) printf("Positive.");
else printf("Not positive.");
```

The compound statement can contain any kind of statements, including another conditional statement:

```c++
if (x > 0) {
  printf("Positive.");
} else {
  if (x < 0) {
    printf("Negative.");
  } else {
    printf("Zero.");
  }
}
```

in this case, both branches of the outer `if` statement contain only a single statement, and so we can rewrite it omitting the curly braces, and sparing an indentation level:

```c++
if (x > 0) printf("Positive.");
else if (x < 0) {
  printf("Negative.");
} else {
  printf("Zero.");
}
```

but again also both branches of the inner `if` statement contain only a single statement, so we can further simplify this to:

```c++
if (x > 0) printf("Positive.");
else if (x < 0) printf("Negative.");
else printf("Zero.");
```

## Functions

A *function* is a set of statements that is given a name, and that can be executed from another place of the code by referring to that name. To define a function we use a *function definition* statement, including the name of the function, and its *body*, i.e. the set of statements to be executed:

```c++
void say_hello() {
  printf("Hello");
}
```

This function can then be executed by placing a function call statement in a place of the source code *after* the function definition:

```c++
void say_hello() {
  printf("Hello");
}

int main() {
  say_hello();
  return 0;
}
```

Had we defined `say_hello` *after* the place where the function is used, which is the `say_hello()` call, the compiler would've generated an error, complaining that it wouldn't be able to locate the `say_hello` function.

A function can return a value; in this case we need to specify the type of the returned value in the function definition, and to add a *return statement* to the body of the function:

```c++
int get_result() {
  return 1;
}

int main() {
  printf("%d", get_result());
}
```

so the call to a function that returns a value is also an expression, which is evaluated to the value returned by the function.

A function that returns no value must use `void` as a replacement for the return value (be aware that `void` is not a type), and contain no `return` statement.

The `return` statement, in addition to produce the value the function can be evaluated to, also marks the end of the execution of the function, and the return of the control to the calling statement, meaning that any additional statements that are placed *after* the return statement will never be executed.

We can still use multiple `return` statements in the same function, for example by returning different values in different branches of a conditional statement:

```c++
char exec() {
  int i { 0 };

  if (i > 0) {
    return 'P';
  }

  return 'N';
}
```

in this case, however, we must be sure that all possible branches of execution terminate with a `return` statement that return a value of the declared type.

We can dynamically control the way a function works by passing parameters, or *arguments* to it. A function must declare what arguments can be passed to it, along with their types:

```c++
void print_number(int number, bool newline) {
  printf("%d", number);
  if (newline) {
    printf("\n");
  }
}

int main() {
  print_number(3, true);
  return 0;
}
```

## The structure of a C++ program

In order to be able to run an executable C++ program, the operating system needs to know at what position of the program the execution should start. This must be defined by the programmer in the source code, by creating a `main` function, representing the starting point of the program, which in its simplest form looks like this:

```c++
int main() {
  // some code
  // some other code
  // ...
  return 0;
}
```

The first line of the `main` function is related to the first actual instruction that will be executed by the operating system when the final executable file will be run.

The `main` function can also control what exit status will be returned to the operating system at the end of the execution, by choosing an integer value to return, which is usually `0` to signal that a correct execution has happened.

For this reason, every C++ program must contain one, and only one, definition of a `main` function.

## Building applications

Given a C++ program represented by a source file like:

```c++
// main.cpp
#include <cstdio>

int main() {
  printf("Hello, world!");
  return 0;
}
```

in order to produce an executable application out of this text file, it needs to be run through the *compiler tool chain*.

There are many different compiler tools available: although they should all abide by the official C++ ISO standard, they might differ for how they optimize the final application, and what additional tools they provide for the developer. A typical compilation command might look like this:

```shell
$ g++ -std=c++20 -fmodules -Wall -Wextra -Werror main.cpp -o app
```

this will produce an executable file named `app`, which we can then invoke in order to run our program:

```shell
$ ./app
Hello, world!
```

When we call the compilation command, the tool chain is launched on the given source files, which will go through three steps: the preprocessor, the compiler and the linker.

The first step of the tool chain is the *preprocessor*, which takes each source file and look for preprocessor directives, like `#include <cstdio>`, and processes them. Preprocessor directives just change the contents of the source files, for example adding new code, or removing some parts of the code. For each file that goes through the preprocessor, a single *translation unit* is produced.

In our example, the `#include <cstdio>` directive instructs the preprocessor to replace the line containing that directive with all the contents of the header file of the `cstdio` library: this is important because that file includes, among other things, the declaration of the `printf` function used some lines of code below.

The next step is the *compiler*, which takes the translation units produced by the preprocessor and produces a single *object file* for each. Object files contain data and instruction in an intermediate format. Instructions can refer to external code that is not in the same object file, by using specific references, like in our example for the external function `printf`, whose behavior is not defined in our program.

The final step is the *linker*, which  takes multiple object files, resolves any external references that are found there so that all references eventually point to actual code, and produces a single executable file. In our example the linker will take the object file produced by the compiler, and the object file already available for the `cstdio` library, which includes the actual definition of the `printf` function, and links them together in order to produce an executable that knows what actual instructions to run when `printf` is called.

When the eventual executable file is executed, the operating system performs the following operations:

- creates a new process with some reserved memory for it
- creates a new set of standard input and output files
- gathers execution arguments
- executes the new process with the collected arguments, starting at the position of the `main` function
- upon process termination, collects the exit status


## TODO

- operators
