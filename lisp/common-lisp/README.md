Lisp refers to a family of programming language, each abiding by a specific set of principles. Despite being hundreds of Lisps in existence, there are two main dialects that are most commonly in use: ANSI Common Lisp and Scheme, where Common Lisp tends to be a bit less consistent and less readable than Scheme, but more powerful.

Common Lisp is just a programming language reference, and implementations exist providing different features: in particular there are implementations providing both native code compilation, and compilation to interpreted bytecode. Two famous Common Lisp implementations are CLISP, which is bytecode compiled, and SBCL, which is instead compiled to native code.

Usually Lisp implementations default to a REPL environment, for example with SBCL:
```
$ sbcl
This is SBCL 1.4.5.debian, an implementation of ANSI Common Lisp.
More information about SBCL is available at <http://www.sbcl.org/>.

SBCL is free software, provided as is, with absolutely no warranty.
It is mostly in the public domain; some portions are provided under
BSD-style licenses. See the CREDITS and COPYING files in the 
distribution for more information.
* _
```

From here we can start typing Common Lisp code:
```
* (+ 3 (* 2 4))

11
```

In the REPL environment expressions are automatically evaluated, and their value is printed on the screen. To exit the REPL environment we can use the `(quit)` function. If an error happens, to get back to the REPL environment, just hit `<ctrl> + d`.

In SBCL, we can run Lisp programs as scripts. If we have the following script file:
```lisp
; test.lisp
(princ (+ 3 (* 2 4)))
```

we can run it from the command line with:
```
$ sbcl --script test.lisp
11
```

Lisp is in general a multi-paradigm language, however it puts an emphasis on functional programming. In the Lisp syntax everything is defined as a list, and lists are written in the following form:
```lisp
(el1 el2 ... eln)
```

The most important construct in lisp is the function call, which is called a *form* in Lisp parlance, and has the following syntax:
```lisp
(name arg1 arg2 ... argn)
```

so the name of the function followed by the list of arguments to call the function with. For example `(* 2 4)` calls the multiplication function, which is named `*`, with the numbers `3` and `4` as arguments.

A Lisp program is always a sequence of function calls: we use functions to define variables, to define new functions, to call functions, etc.

The arguments of a form can be of different types. The most important ones are: symbols, numbers and strings, in addition of course to other lists.

Symbols are identifiers found in the code, and they are used to refer to variables, functions, and actual symbols. Actual symbols are identifiers to which no definition is attached, because they just carry a meaning that is valuable for the programmer only. In Lisp all identifiers are case-insensitive, thus Lisp internally stores identifiers as all uppercase.

Numbers can be floating-point or integers: floats are those written with a decimal point in them. Numbers don't need any special syntax to be identified, because all identifiers that just look like numbers are treated as numbers: this also means that we can't name variables, functions and symbols as just numbers.

Strings are identified as usual by enclosing them between double quotes.

Lisp code is executed by default in *code mode*, meaning that form arguments are always evaluated. Numbers and strings are of course evaluated to themselves, being them raw values. Symbols are evaluated differently according to their type: variables are evaluated to the value they're attached to; functions are evaluated only if they are the first element of a form, or if they're explicitly evaluated with the `function` function, or with the lambda syntax `#'`; actual symbols are never evaluated. Finally, lists are evaluated only if they are forms, or if they're part of special constructs, like function parameters.

To be able to use actual symbols, we need to switch to *data mode*, where Lisp doesn't try to evaluate the code it finds. For example the identifier `princ`, if used in code mode, can be a function if it's used as the first element of a form, like in `(princ 1)`, or a variable, if used in any other way, like `(princ princ)`, which would print the contents of the `princ` variable (assuming it has been defined). Instead, `'princ` represents the `princ` actual symbol, which is not evaluated, because the apostrophe `'` caused Lisp to enter data mode, thus treating `princ` as just data, instead of code to be executed.

Data mode doesn't work with numbers and strings, since they're already data (this means that `'1` is exactly equivalent to `1`). We can use code mode to create lambdas, thus to pass functions as arguments, with the following syntax: `#'princ`: the `princ` thus passed will be treated as a function that can be executed by the callee; this syntax is equivalent to `(function princ)`, actually evaluating `princ` as a function.

Data mode is also used to refer to raw lists, like `'(1 2 3)`: in fact lists in code mode are always evaluated, and what happens depends on the context, for example the list can be a form, or the syntax required for the argument of a form, like function parameters.

We can embed Lisp code in data mode using *quasiquoting*, which works much like string interpolation:
```lisp
`(there is a ,(caddr edge) going ,(cadr edge) from here)
```

here, quasiquoting is started with the backtick operator ```, after which we are in data mode: to embed some Lisp code from here, we use the comma operator `,`, after which we can insert some Lisp code, like a form or a variable to be evaluated; the embedded code mode ends with the end of the form, after which we are in data mode again, so that we need to enter code mode again with `,` if we want to embed another form or variable.

Every list in Lisp is based on the concept of *cons cells*: a cons cell is a couple of a value and a pointer, pointing to another cons cell, much like a linked list works. For example, `'(1 2 3)` is actually made of a cons cell containing the number `1` and a pointer to another cons cell, containing the number `2`, and a pointer to another cons cell containing `3` and the empty list `()` (which can also be referred to as `nil`).

The `cons` function creates a cons cell given two values, for example:
```
* (cons 1 2)

(1 . 2)
```

where `(1 . 2)` is a special syntax used only to represent cons cells, and does not represent a list. We can use the empty list as a pointer:
```
* (cons 'data ())

(DATA)
```

where we see that the empty list is not displayed, since it's just a way to indicate that this cons cell doesn't point to any other cell. Actually, this result is also an actual list, because it's just like a chain of cons cell (made of just one cons cell, which is also the terminating one), which is exactly what a list is.

This means that when we create a list, like `'(1 2 3)`, we are really just using syntactic sugar for the following:
```lisp
(cons 1 (cons 2 (cons 3 ())))
```

Lisp provides the `list` function that does exactly this:
```
* (list 1 2 3)

(1 2 3)
```

Lisp supports global variables, to define which we can use the `defparameter` function:
```
* (defparameter *small* 1)

*SMALL*
* *small*

1
* (defparameter *big* 100)

*BIG*
* *big*

100
```

Here we're putting *earmuffs* around global variables names: this is just a convention in the Lisp community, to identify global variables among other kinds of identifiers. Actually, asterisks are perfectly legal characters to be used in identifiers, so Lisp will just consider them as part of the identifier, and not interpret them in any special way.

In Lisp, identifiers are case-insensitive, and thus Lisp stores them always as uppercase: this is why they're printed back all uppercase. As we can see, the `defparameter` function returns the name of the variable that has just been defined. We can print the contents of a global variable by just inserting its name in the REPL, without surrounding it with parenthesis; this of course won't work from a script file.

Global variables defined with `defparameter` are *not* immutable, meaning that they can be overwritten:
```lisp
* (defparameter *foo* 5)

*FOO*
* *foo*

5
* (defparameter *foo* 6)

*FOO*
* *foo*

6
```

To define immutable global variables, we use the `defvar` function:
```
* (defvar *foo* 5)

*FOO*
* *foo*

5
* (defvar *foo* 6)

*FOO*
* *foo*

5
```

However, `defvar` won't throw any error when trying to modify an immutable variable: it will just fail silently.

To change the value of a mutable global variable, we use `setf`:
```
* *big*

100
* (setf *big* 4)

4
* *big*

4
```

Of course we could've just re-defined `*big*` here, but that's not the way it's intended to be used.

To define global functions, we use the `defun` function, which takes the name of the function, and the list of its arguments:
```
* (defun guess-my-number ()
    (ash (+ *small* *big*) -1))

GUESS-MY-NUMBER
```

as usual, `defun` returns the name of the function that has just been defined.

Functions defined with `defun` can be re-defined, just by calling again `defun` with the same function name; however, `defun` cannot be used to re-define core Lisp functions: for example, we cannot re-define `defun`.

We can create a local context with the `let` function, where a list of variables that will be defined are visible:
```
* (let ((a 5)
        (b 6))
    (+ a b))

11
```

So `let` takes a list of variables definitions, and a body, which is an expression that can use those variables. The value of the expression is also returned by the `let` function. Of course the variables defined inside `let` are not visible from the outside of the context created by `let`.

Similarly, we can define a context where only the defined functions are visible, with the `flet` function:
```
* (flet ((f (n)
           (+ n 10))
         (g (n)
           (- n 3)))
    (g (f 5))

12
```

of course the two functions `test1` and `test2` will only be visible in the last expression.

The `flet` function doesn't allow functions that are being defined, to refer to themselves. To allow this, we use the `labels` function:
```lisp
* (labels ((a (n)
             (+ n 5))
           (b (n)
             (+ (a n) 6)))
    (b 10))

21
```

this of course can also be used to provide recursion.

The `cons` function, in addition to create cons cells, can also be used to append an element to the front of a list:
```
* (cons 1 '(2 3))

(1 2 3)
```

The `car` function is used to get the first element of the cons cell representing a list, which also means the first element of the list:
```
* (car '(1 2 3))

1
```

while the `cdr` function is used to get the second element of the cons cell representing a list, which, being the pointer to the second cons cell, also represents the tail of the list, from the second to the last element:
```
* (cdr '(1 2 3))

(2 3)
```

Lisp provides short hand functions to combine together `car` and `cdr` in different ways, for example:
- `(car (cdr '(1 2 3)))` first gets the tail `'(2 3)`, and then gets the first element of it `2`. Lisp provides the `cadr` function for this, that thus retrieves the second element of a list.
- `(car (cdr (cdr '(1 2 3))))` first gets the tail `'(2 3)`, then gets the tail of it `'(3)`, and then gets the first element of it `3`. Lisp provides the `caddr` function for this, that thus retrieves the third element of a list.
- `(cdr (car '((1 2) (3 4))))` first gets the head `'(1 2)`, and then gets the tail of it `'(2)`. Lisp provides the `cdar` function for this.
- ...

The `if` function is used to create a conditional structure, and has the following syntax: `* (if condition true-value false-value)`, which means that if the given boolean `condition` is evaluated to true, `true-value` is returned, otherwise `false-value` is returned. For example:
```
* (if (= 1 2) 'hey 'joe)

JOE
```

The most simple boolean expressions are `()` or `nil`, representing false, and any non-empty list, representing true, so that `(if nil 'hey 'joe)` will return `JOE`, and `(if (1) 'hey 'joe)` will return `HEY`. Instead of using a non-empty list, we also have available the value `t`, which much like `nil` represents true (also, like `nil`, it doesn't need to be escaped in data mode like `'t` because Lisp is designed to recognize `t` and `nil` as data already).

In both functions and conditionals, we can only execute a single expression:
```lisp
(defun myfunction (a b)
  (somefun a b))
```

here the body of the function `myfunction` can only be constituted of a single expression; similarly:
```lisp
(if mycondition
  (somefunction 1 2)
  (otherfunction 3 4))
```

here, whether the condition is true or false, whichever branch is taken, it can still only be constituted of a single expression.

To execute more than one expression sequentially, we can use the `progn` function:
```
* (progn
  (print 'using)
  (print 'multiple)
  (print 'expressions))

USING
MULTIPLE
EXPRESSIONS
EXPRESSIONS
```

`progn` executes sequentially all its given arguments as expressions, and returns the value of the last one.

A couple of alternatives to `if` are provided that already support execution of multiple sequential expressions:
```lisp
(when (= 1 1)
      (print 'using)
      (print 'multiple)
      (print 'expressions))
```
```
(unless (= 1 2)
        (print 'using)
        (print 'multiple)
        (print 'expressions))
```

Additionally, the `cond` function allows to stack multiple conditions with multiple expressions, much like stacking *if*'s.
```lisp
(cond ((eq person 'henry)  (print 'found)
                           (print 'first))
      ((eq person 'johnny) (print 'found)
                           (print 'second))
      (t                   (print 'generic)
                           (print 'case)
```

here all branches where the condition matches will be executed. Putting a `t` condition last, we're sure that at least that one will always be executed.

The `case` function works much like `cond`, where all conditions are `eq` ones:
```lisp
(case person
      ((henry)   (print 'found)
                 (print 'henry))
      ((johnny)  (print 'found)
                 (print 'johnny))
      (otherwise (print 'found)
                 (print 'unknown))
```

here we use the value `otherwise` to match all remaining cases. Notice how the cases matched to are lists of symbols, and not just single symbols: this allows to execute the same expressions for more than one match.

Many Lisp functions accepts additional parameters at the end of the arguments list, to enable special behaviors. For example, the `find` function accepts a *keyword parameter*, allowing to better specify the position of the item to find:
```
* (find 'y '((5 x) (3 y) (7 z)) :key #'cadr)

(3 Y)
```

here, starting from the first list of the given list of lists, the `cadr` function is applied to return the second item, and then the resulting value is compared to the given `'y`. If they are identical, the list where the match was found is returned.
