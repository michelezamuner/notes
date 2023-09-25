# C

To access the C compiler in Unix-like system, we use the `cc` program:

```shell
$ cc --version
Apple clang version 14.0.0 (clang-1400.0.29.202)
Target: x86_64-apple-darwin21.6.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```

In order to compile a C program, we can use the default settings of `make`. Let's say we have an `ex1.c` source file:

```shell
$ make ex1
cc ex1.c -o ex1
```

We can pass compiler specific flags with:

```shell
$ CFLAGS="-Wall" make ex1
cc -Wall ex1.c -o ex1
```

This would produce a `ex1` executable we can run:

```shell
$ ./ex1
You are 100 miles away.
```

## Compiler

- `-Wall`: show all warnings
- `-g`: toggle debugging

## Debugger

For C programs one of the best solutions for debugging is `lldb`. In order to debug a program, we first need to load it into `lldb`:

```shell
$ lldb ex1
(lldb) target create "ex1"
Current executable set to '/Users/michele/Projects/_tmp/c/learn-c/ex1' (x86_64).
(lldb)
```

at this point `lldb` is ready to receive commands to debug the loaded program.

## Makefile

```makefile
CFLAGS=-Wall -g

all:
	make ex1

clean:
	rm -f ex1
```

## References

- "Learn C the hard way" Shaw, Addison-Wesley 2016
