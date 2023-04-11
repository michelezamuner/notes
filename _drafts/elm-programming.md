# Elm Programming

Elm is a functional programming language that compiles to JavaScript, designed to build frontend Web applications.

## Developing with Elm

Elm can be installed as a Node package:
```shell
$ npm install -g elm
$ elm --version
0.19.1
```

We can use a REPL environment for Elm:
```shell
$ elm repl
> 10 + 20
30 : number
> :exit
```

To bootstrap a project:
```shell
$ elm init
```

This will create a `elm.json` configuration file with some default dependencies, including `elm/html` to handle the DOM. Then with:
```shell
$ elm make
```

we download the dependencies inside the `elm-stuff` folder.

There are some conventions that are enforced by the Elm compiler regarding the project's file structure. The most simple project will contain a `src/Main.elm` file, defining the `Main` module of the project:
```elm
module Main exposing (..)

import Html exposing (text)

main =
  text "Hello, world!"
```

At this point we need to compile our program:
```shell
$ elm make src/Main.elm
```

This will create a new `index.html` file at the root of the project. Opening this HTML file in a browser we'll see an HTML page containing the simple text `Hello, world!`.

The `Main` module must define a `main` function. In our example, we're using the `text` function from the `Html` module to print a string on the Web page.

Elm supports inline comments and multiline comments:
```elm
-- this is a single line comment

{- This is a
  Multi-line comment
-}
```

## Modules

Any code in an Elm program must be contained within a module, and every source file defines a module, which must have the same name as the file. The first line of the file declares the module:
```elm
module Variables exposing (..)
```

Here we're declaring that the current file contains a module called `Variables`. The `exposing` keyword tells the compiler which identifiers from the module will be accessible from the other parts of the program. The `(..)` syntax means "all identifiers".

We can load specific modules in the REPL, in order to use them during live coding:
```shell
$ elm repl
> import Main exposing (..)
```

We can import a whole module with:
```elm
import String exposing (..)
-- ...
fromInt(3)
```

Or we can import specific identifiers from a module:
```elm
import String exposing (fromInt)
-- ...
fromInt(3)
```

Or again we can directly refer to the module in the source code without importing it:
```elm
String.fromInt(3)
```

## Types

Elm supports the following numeric types:
- `number`: represents any numeric value, like `7`
- `Float`: represents floating point values, like `3.5`
- `Int`: represents integer values, like `3`

Strings can be represented with literal notation, like `"Hello, world!"`, and are of type `String`. Characters can be represented with literal notation, like `'T'`, and are of type `Char`.

Useful string functions from the `String` module are:
- `isEmpty : String -> Bool`: tells if a string is empty
- `reverse : String -> String`: reverses a string
- `length : String -> Int`: returns the number of characters of the string
- `append : String -> String -> String`: append a string to another
- `concat : List String -> String`: concatenates the elements of a list of strings
- `split: String -> String -> List String`: splits a string given a separator
- `slice: Int -> Int -> String -> String`: produces a substring given the start index, the end index and the original string
- `contains: String -> String -> Bool`: tells if a string contains a substring
- `toInt: String -> Result.Result String Int`: parses a string to an integer
- `fromInt: Int -> String`: parses an integer to a string
- `toFloat: String -> Result.Result String Float`: parses a string to a float
- `fromFloat : Float String`: parses a float to a string
- `fromChar : Char -> String`: creates a string from a character
- `toList : String -> List Char`: converts a string to a list of characters
- `fromList: List Char -> String`: converts a list of characters to a string
- `toUpper: String -> String`: converts a string to uppercase
- `trim : String -> String`: removes whitespace from the beginning and end of a string
- `filter: (Char -> Bool) -> String -> String`: filters the characters of a string according to the given function
- `map: (Char -> Char) -> String -> String`: maps every character of a string according to the given function

Elm supports the `Bool` type to represent the boolean values `True` and `False`.

We can define type aliases with:
```elm
type alias MyString = String

myString : MyString
myString = "My string"
```

We can define union types with:
```elm
type PaymentMode = CreditCard|NetBanking|DebitCard
```

## Variables

Variables are declared by just introducing their identifier:
```elm
message:String
message = "Variables can have types in Elm"
```

Alternatively, we can omit the type annotation, when the compiler can figure out the type by itself by looking at the initialization value:
```elm
message = "Variables can have types in Elm"
```

## Data structures

A list in Elm is an immutable collection of values of the same type. Lists are defined literally by declaring their elements within square brackets:
```elm
list: List number
list = [1, 2, 3, 4]
```

Lists are immutable in Elm, so any function that works with lists will always return a new list, and never mutate existing ones.

These are useful functions from the `List` module:
- `isEmpty : List a -> Bool`: tells if a list is empty
- `reverse : List a -> List a`: reverses a list
- `length : List a -> Int`: returns the length of a list
- `maximum : List comparable -> Maybe.Maybe comparable`: returns the maximum value of a list
- `minimum : List comparable -> Maybe.Maybe comparable`: returns the minimum value of a list
- `sum : List number -> number`: returns the sum of all elements of the list
- `product : List number -> number`: returns the product of all elements of the list
- `sort : List comparable -> List comparable`: sorts a list
- `concat : List (List a) -> List a`: merges a list of lists
- `append: List a -> List a -> List a`: merges two lists
- `range: Int -> Int -> List Int`: produces a range from one number to another
- `filter: (a -> Bool) -> List a -> List a`: filters a list according to a given function
- `head: List a -> Maybe.Maybe a`: returns the first element of a list
- `tail : List a -> Maybe.Maybe (List a)`: returns the tail of a list

The cons operator `::` appends an element to the front of a list:
```elm
10::[20, 30, 40, 50]
-- [10, 20, 30, 40, 50] : List Int
```

A tuple in Elm is an immutable collection of values of different types:
```elm
tuple: ( number, String, List Int )
tuple = (1, "a", [1, 2, 3])
```

Some tuple functions only work with tuples of exactly two elements, i.e. couples.

These are useful functions from the `Tuple` module:
- `first : ( a1, a2 ) -> a1`: gets the first element of a couple
- `second : ( a1, a2 ) -> a2`: gets the second element of a couple

E record in Elm is a composite immutable data structure, made of different named fields:
```elm
company : { name : String, rating : Float }
company = { name = "Company", rating = 3.5 }
```

We can access record's fields with:
```elm
company.name
-- "Company" : String
```

We can update an existing record with:
```elm
{ company | name = "Other name", rating = 2.5 }
-- { name = "Other name", rating = 2.5 } : { name : String, rating : Float }
```

Since records are immutable, this will of course produce a new record, and leave untouched the original one.

It's common to use type aliases to give a meaningful name to a record type:
```elm
type alias Developer = { name : String, location : String, age : Int }
dev1 : Developer
dev1 = { name = "Kannan", location = "Mumbai", age = 20 }
```

The type alias can be used as a constructor:
```elm
dev1 : Developer
dev1 = Developer "Kannan" "Mumbai" 20
```

## Let structure

The `let` structure is used to handle complex variable definitions:
```elm
result: Int
result =
  let
    (first, _, third) = (10, 20, 30)
  in
    first + third
```

Here we use "list destructuring" to read the first and third elements of the given list in the `let` block, and then we produce the value that should be returned by the expression in the `in` block.

## Functions

In Elm functions are defined by providing a declaration and an implementation:
```elm
hello: () -> String
hello _ = "Hello, world!"
```

Here `()` refers to the absence of a value in the declaration, and `_` refers to the absence of a value in the definition. To call a function:
```elm
hello
-- "Hello, world!" : String
```

Functions in Elm must always produce a return value, since everything in Elm must be able to be evaluated as an expression.

Functions that take arguments are defined like this:
```elm
sum: Int -> Int -> Int
sum a b = a + b
```

and used like this:
```elm
sum 1 2
-- 3
```

Functions can be piped to one another with the pipe operator `|>`:
```elm
["a", "b", "c", "d", "e", "f"] |> List.reverse |> String.join "-"
```

## Conditional structure

Elm supports the `if` conditional structure:
```elm
if 10 > 5 then "10 is bigger" else "10 is smaller"
```

The `if` structure is itself an expression, meaning that it is evaluated by the value of the executed branch:
```elm
result = if 10 > 5 then "hey" else "joe"
-- "hey" : String
```

Elm also supports the `case` statement:
```elm
case n of
  0 -> "n is Zero"
  _ -> "n is not Zero"
```

where the `_` placeholder represent the default branch.

## Iterations

Elm does not have iterative structure, rather it always uses recursion to perform repetitive executions:
```elm
sayHello : Int -> String
sayHello n =
  case n of
    1 -> "Hello:1 "
    _ -> "Hello:" ++ String.fromInt n " " ++ sayHello (n - 1)
```

## Options and errors

Elm does not provide any `null`-like value. Instead, to represent the absence of a value it uses an union type called `Maybe T`, that can represent either the presence of a value (`Just T`), or the absence of any value (`Nothing`):
```elm
userName : Maybe String
userName : Just "John"

userSalary : Maybe Float
userSalary : Nothing
```

So `Maybe` is a generic type taking one type parameter, which contains the type of the value, to be used in case a value actually exists. `Nothing` doesn't need any type specification, since the absence of a value is always the same concept, regardless of types.

Similarly, Elm does not provide any exceptions mechanism. Rather, to handle error situations it uses the `Result E T` type, which is another generic union type that can represent a successful result (`Ok T`) or an error (`Err E`):
```elm
userId : Result String Int
userId = Ok 10

emailId : Result String Int
emailId = Err "Not valid emailId"
```

## Elm architecture

Elm uses a standard architecture to build frontend applications, based on these concepts:
- *models*: entities representing the application state
- *views*: the output presented to the user, for example an HTML text, that can also be used to generate inputs
- *messages*: produced when the user interacts with the view
- *updates*: the handlers of the user inputs, which usually updates models
- *commands*: communication of the application with other components, like external services
- *subscriptions*: communication of other components with the application

The `Browser` module of Elm provides a `sandbox` function which is helpful to create simple applications based on the Elm architecture:
```elm
-- ...
import Browser exposing (sandbox)
-- ...
main =
  sandbox
    {
      init = model
      ,update = update
      ,view = view
    }
```

Here the `model` is can be just a variable:
```elm
model = 0
```

`update` is the function that updates the model according the user input (the message):
```elm
type Message = Add | Subtract
update msg m =
  case msg of
    Add -> m + 1
    Subtract -> m - 1
```

and `view` is the function that produces the output by reacting to changes to the model:
```elm
-- ...
import Html exposing (h1, div, text, button)
import Html.Events exposing (onClick)
-- ...
view m =
  div[][
    h1[][text "Counter App"]
    ,div[][
      button[onClick Subtract][text "-"]
      ,span[][text (String.fromInt m)]
      ,button[onClick Add][text "+"]
    ]
  ]
```

Here the `sandbox` application will first render the `view` passing it the `model` as argument (which is `0` at the beginning), and waits for user's actions. When the user click either button, a `Message` is sent to the `update` function, along with the same `model`: the `update` function will return a new value that should be used as `model`, and the `view` is automatically rendered again with the new value of the `model`.
