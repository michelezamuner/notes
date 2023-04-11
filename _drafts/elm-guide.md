# Elm guide

Elm is a functional language that compiles to JavaScript, designed to produce Web browser applications.

Data in Elm is represented by typed values. Values are transformed by functions:
```elm
greet name =
  "Hello " ++ name ++ "!"
```

Elm support conditional expressions with:
```elm
if name == "Abraham Lincoln" then
  "Greetings Mr. President!"
else
  "Hey!"
```

The most simple composite type in Elm is the list, which is a collection of elements of the same type:
```elm
names = [ "Alice", "Bob", "Chuck" ]
-- [ "Alice", "Bob", "Chuck" ]
```

Lists in Elm do not support indexing (i.e. random access), but they need to be iterated over.

some useful functions provided by the `List` module are:
- `List.isEmpty list`: tells if a list is empty
- `List.length list`: gets the length of a list
- `List.reverse list`: produces a new list with elements in reverse order
- `List.sort list`: produces a new list with elements sorted
- `List.map function list`: produces a new list with elements mapped with the given function

The most simple composite type that supports elements of different types is the tuple, which is a collection of two or three elements of any type:
```elm
result = (True, "name accepted!")
-- (True, "name accepted!")
```

We can access the first element of a tuple with `Tuple.first`:
```elm
Tuple.first (1, 2, 3)
-- 1
```

If the tuple is a couple, we can use `Tuple.second`:
```elm
Tuple.second (1, 2)
-- 2
```

To represent composite data with different types and any number of elements we can use records, which are collections of named elements:
```elm
john =
  { first = "John"
  , last = "Hobson"
  , age = 81
  }
-- { first = "John", last = "Hobson", age = 81 }
```

we can access records elements with the dot operator:
```elm
john.last
-- "Hobson"
```

Access to records elements can also be rewritten as a function instead of a property, with field access functions:
```elm
.last john
-- "Hobson"
```

We can update records, meaning creating new records from existing ones by updating some fields:
```elm
{ john | last = "Adams"}
-- { first = "John", last = "Adams", age = 81 }
```

## The Elm architecture

An Elm program handles some data, and produces some HTML to present the results to the end user on their browser. The user then interacts with this HTML by sending messages back to the application, for example by clicking a button.

In Elm, the data is represented by *models*, the HTML presentation is made of *views*, and the reaction to user inputs are *updates*.

Any Elm program needs to have a `Main` module exposing a `main` function:
```elm
module Main exposing (..)
-- ...
main =
  -- ...
```

the `main` function take no argument and returns no value.

We can bootstrap a basic Elm architecture using `Browser.sandbox`:
```elm
module Main exposing (..)

import Browser

main =
  Browser.sandbox { init = init, update = update, view = view }
```

`Browser.sandbox` takes a record with fields `init`, `update` and `view`.

The `init` field is used to initialize our models. For example it can be a model itself:
```elm
-- ...
type alias Model = Int

init : Model
init =
  0
-- ...
```

The `update` field defines the function that needs to be called when a message is sent by the user:
```elm
type Msg = Increment | Decrement

update : Msg -> Model -> Model
update msg model =
  case msg of
    Increment ->
      model + 1
    Decrement ->
      model - 1
```

The `sandbox` method that creates the application passes the message that comes from the user to the `update` function, along with the model defined in the `init` field. The return value of the `update` function is then used as a new value for the model defined in `init`.

Last, the `view` field defines the HTML that needs to be presented to the user:
```elm
view: Model -> Html Msg
view model =
  div []
    [ button [ onClick Decrement ] [ text "-" ]
    , div [] [ text (String.fromInt model ) ]
    , button [ onClick Increment ] [ text "+" ]
    ]
```

After `update` is called and `init` is updated, the output HTML needs to be updated as well: to do this, Elm calls the `view` function passing the current model stored in `init`. This function returns an `Html` object that is parameterized to our custom message type `Msg`, that is used to produce the final HTML. Here we see that when the user clicks on the buttons a message `Decrement` or `Increment` is produced, which is what's actually passed to the `update` function.
