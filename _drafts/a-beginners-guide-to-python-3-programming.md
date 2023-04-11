# A Beginners Guide to Python 3 Programming

## Statements and expressions

A *statement* in Python is an instruction that can be executed by the interpreter, for example:
```python
print('Hello, World!')
```

Here we have a statement composed by an instruction that is a call to the `print()` function. The main instruction of a statement can also contain other nested instructions, for example:
```python
name = input('Enter your name: ')
```

Here we still have a single statement, defined by the assignment instruction, but the right-hand side of the assignment is in turn composed of another nested instruction: a call to the `input()` function.

An *expression* in Python is an instruction that produces a value. For example:
```python
4 + 5
```

this sum instruction will produce a value, which is the result of the specific sum that has been written. Also:
```python
input('Enter your name: ')
```

is an expression, because it produces a string value which is the text that has been input by the user.

## Variables

Python is a *dynamically typed* language, meaning that the type of variables can change during the execution of a program. For example:
```python
a = 'Hello'
print(a) # output: 'Hello'
a = 42
print(a) # output: 42
```

Here we first defined the `a` variable to contain a string, `'Hello'`, but then we assigned a number, `42`, to the same variable `a`.

To find out the type of a variable, we can use the `type()` function:
```python
my_variable =  'Bob'
print(type(my_variable)) # output: <class 'str'>
```

If we try to use a variable that has not been defined (for example that has not been initialized), we'll receive an error like:
```
NameError: name 'count_number_of_tries' is not defined
```

To access a global variable from inside a function, we need to use the `global` keyword:
```python
max = 100
def print_max():
  global max
  max = max + 1
  print(max)
```

otherwise the Python interpreter would think that we're referring to a local `max` variable, which hasn't been defined though, thus resulting in an error.

To access a variable defined locally within a function, from another function also defined locally within that function, we need to use the `nonlocal` keyword:
```python
def outer():
  title = 'original title'
  def inner():
    nonlocal title
    title = 'another title'
```

The lifetime of variables which are defined locally inside functions is as long as the function executes, and they are destroyed as soon as the function returns or terminates.

## Strings

A string is an ordered sequence of characters. Strings are also immutable, meaning that any string manipulation operation will in fact produce a new string object, leaving the original one untouched. Strings can be defined by either single quotes `'` or double quotes `"`, with no difference between the two: they are both supported to help write strings containing quotes.

Strings can also be defined by triple quotes `'''`, which allow to define multi-line strings:
```python
z = '''
Hello
World
'''
```

These are some of the most common operations that can be performed with strings:
```python
'Good' + " day" # Concatenation, produces: 'Good day' (string)
len('Hello, World!') # Length (number of characters), produces: 13 (int)
'Hello'[3] # Indexed access, produces: 'l' (string)
'Hello'[1:3] # Substring, produces: 'ell' (string)
'Hello'[:3] # Substring from the start, produces: 'Hell' (string)
'Hello'[3:] # Substring to the end, produces: 'lo' (string)
'Hi' * 5 # Repetition, produces: 'HiHiHiHiHi' (string)
'Hey, Joe'.split(' ') # Split, produces: ['Hey,', 'Joe'] (list)
'A B C'.count(' ') # Occurrences count, produces: 2 (int)
'Hey, Joe'.replace('Joe', 'Bob') # Replacement, produces: 'Hey, Bob' (string)
'Hey, Joe'.find('Joe') # Index of substring, produces: 5 (int)
'Hey, Joe'.find('Bob') # Index of substring (no result), produces: -1 (int)
str(12) # Casting, produces: '12' (string)
'Hey' == 'Joe' # Comparison for equality, produces: True (bool)
'Hey' != 'Joe' # Comparison for inequality, produces: False (bool)
'Hey'.startswith('H') # Check of starting character, produces: True (bool)
'Joe'.endswith('H') # Check of ending character, produces: False (bool)
'Hey, Joe'.istitle() # Check if the string has the form of a title, produces: True (bool)
'HEY'.isupper() # Check if all string's characters are uppercase, produces: True (bool)
'HEY'.islower() # Check if all string's characters are lowercase, produces: False (bool)
'Hey, Joe'.isalpha() # Check if all string's characters are alphabetic, produces: False (bool)
'123'.isnumeric() # Check if all string's characters are numeric, produces: True (bool)
'Hey'.upper() # Converts to uppercase, produces: 'HEY' (string)
'HEY'.lower() # Converts to lowercase, produces: 'hey' (string)
'hey, joe'.title() # Converts to title, produces: 'Hey, Joe' (string)
'Hey, Joe'.swapcase() # Converts uppercase characters to lowercase, and vice-versa, produces: 'hEY, jOE' (string)
'  A  '.strip() # Trims leading and trailing spaces, produces: 'A' (string)
```

The concatenation operator `+` does no automatic type conversion, so if we want to concatenate a string with an integer, we need to cast the integer to string first, like `'Number' + str(21)`.

Strings can be formatted with the `format()` method, taking values to be used in place of the placeholders contained in the format string:
```python
'{} is {} years old'.format('Adam', 20) # Produces: 'Adam is 20 years old'
```

so the `format()` method also casts values to string before applying them. We can add an index to the placeholder to refer to the order format arguments are passed in:
```python
'Hello {1} {0}, you got {2}%'.format('Smith', 'Carol', 75) # Produces: 'Hello Carol Smith, you got 75%'
```

We can also use named placeholders.
```python
'{artist} sang {song} in {year}'.format(artist='Paloma Faith', song='Guilty', year=2017)
# Produces: 'Paloma Faith sang Guilty in 2017'
```

We can do more sophisticated string formatting as well::
```python
'|{:25}|'.format('25 characters width') # Minimum placeholder width, produces: '|25 characters width      |'
'|{:10}|'.format('10 characters width') # Minimum placeholder width, produces: '|10 characters width|'
'|{:<25|}'.format('left aligned') # Left alignment (default), produces: '|left aligned             |'
'|{:>25|}'.format('right aligned') # Right alignment, produces: '|          right alignment|'
'|{:^25|}'.format('centered') # Centered alignment, produces: '|         centered        |'
'{:,}'.format(1234567890) # Thousands separator, produces: '1,234,567,890'
```

An alternative to format strings is to create string templates using the `strings.Template()` function, taking the format string where placeholders are defined with the dollar sign `$`. Once we have the template, we can apply it to different sets of values with the `substitute()` method, to produce different strings following the same template:
```python
import string
template = string.Template('$artist sang $song in $year')
template.substitute(artist='Freddie Mercury', song='The Great Pretender', year=1987)
# Produces: 'Freddie Mercury sang The Great Pretender in 1987'
template.substitute(artist='Ed Sheeran', song='Galway Girl', year=2017)
# Produces: 'Ed Sheeran sang Galway Girl in 2017'
```

We can also pass the set of replacements with a dictionary:
```python
template.substitute(dict(artist = 'Billy Idol', song = 'Eyes Without a Face', year = 1984))
# Produces: 'Billy Idol sang Eyes Without a Face in 1984'
```

To include an actual dollar sign in a template string, we can use the double dollar sign `$$`, like in `'Price: $price$$'` to produce strings like `'Price: 190.0$'`. We can surround the template variable between curly braces to concatenate it to other characters, like in `'${noun}inification'`.

The `substitute()` method will throw an error if template variables are not all provided with a value. The `safe_substitute()` template method will substitute only the template variables that are provided, leaving the others untouched, like in:
```python
template.safe_substitute(artist='David Bowie', song='Rebel Rebel')
# Produces: 'David Bowie sang Rebel Rebel in $year'
```

## Numbers, booleans and None

Python has a single integer type, which is `int`for integers of all sizes, so it's not necessary to rely on different types to represent very big numbers for example.

We can convert a value to an integer using the `int()`function, for example:
```python
int('100') # Produces: 100
int(1.0) # Produces: 1
```

We can convert a value to a float number using the `float()` function, for example:
```python
float(1) # Produces: 1.0
float('1.5') # Produces: 1.5
```

We can convert a value to a boolean with the `bool()` function, for example:
```python
bool(1) # Produces: True
bool(0) # Produces: False
bool('any non-empty string') # Produces: True
bool('') # Produces: False
```

In Python booleans are actually a subclass of integers, so we can also cast booleans to integers:
```python
int(True) # Produces: 1
int(False) # Produces: 0
```

The special value `None`, representing the absence of any value, is of the special type `NoneType`. This can be used to initialize variables to no value:
```python
winner = None
```

We can then check if a variable has no value with:
```python
winner is None # Produces: True
winner is not None # Prouces: False
```

## Conditionals

The most basic conditional statement in Python is the `if/else/elif` statement:
```python
if savings == 0:
  print('Sorry, no savings')
elif savings < 500:
  print('Well done')
else:
  print('Thank you')
```

Python also provides `if` expressions:
```python
status = ('teenager' if age > 12 and age < 20 else 'not teenager')
```

The conditions that are used within the `if` construct should be of boolean type, or convertible to boolean. In particular:
- `0`, empty strings and `None` equate to `False`
- non-zero numbers, non empty strings and any object equate to `True`

## Loops

Python provides the standard `while` loop statement:
```python
count = 0
while count < 10:
  print(count)
  count++
```

and the standard `for` statement:
```python
for i in range(0, 10):
  print(i)
```

We can use `break` to break out of a loop earlier:
```python
for i in range(0, 6):
  if i == 3:
    break
  ...
```

and `continue` to skip to the next iteration:
```python
for i in range(0, 10):
  print(0, ' ', end=''):
  if i % 2 == 1:
    continue
  ...
```

In Python a `for` loop can have an `else`block at the end of the loop, which will be executed if and only if the loop could run through all iterations; in other words, if a `break` condition is triggered in the loop, making it end prematurely, the `else` block won't be executed:
```python
print('Only print code if all iterations completed')
num = int(input('Enter a number to check for: '))
for i in range(0, 6):
  if i == num:
    break
  print(i, ' ', end='')
else:
  print()
  print('All iterations successful')
```

## Functions

The basic syntax of a function definition in Python is:
```python
def function_name(parameter, list):
  """docstring"""
  statement
  ...
  return value
```

A function can return multiple values, and we can assign them to multiple variables at the same time:
```python
def swap(a, b):
  return b, a

x, y = swap(a, b)
```

What happens here is that the function returns a `tuple`, which supports the multiple assignment syntax.

The docstring defined in the function can be retrieved from the code with:
```python
function_name.__doc__
```

which returns a string containing the actual docstring.

Function parameters can have default values:
```python
def greeter(name, message = 'Live Long and Prosper'):
  ...

greeter('Eloise')
greeter('Eloise', 'Hope you like Python')
```

Optional parameters (those with a default value) must all be the last ones in the parameter list, meaning that once a parameter is defined with a default value, all next parameters must have a default value as well.

Functions invocation support named arguments, meaning that we can refer to a specific parameter by its name, instead of by its position:
```python
greeter('Eloise', message = 'Hope you like Python')
greeter(name = 'Eloise', message = 'Hope you like Python')
```

However, we cannot place positional arguments after named ones. For example this:
```python
greeter(name = 'Eloise', 'Hope you like Python') # ERROR!
```

will produce an error.

Functions support arbitrary arguments:
```python
def greeter(*args):
  for name in args:
    print('Welcome', name)

greeter('John', 'Denise', 'Phoebe', 'Adam', 'Gryff', 'Jasmine')
```

The `*args` syntax works only with positional arguments. To have access to an arbitrary number of named arguments, we use the `**kwargs` syntax:
```python
def my_function(*args, **kwargs):
  for arg in args:
    print('arg:', arg)
  for key in kwargs.keys():
    print('key:', key, 'has value:', kwargs[key])

my_function('John', 'Denise', daughter='Phoebe', son='Adam')
```

An anonymous, or lambda, function is defined as:
```python
lambda i : i * i
```

The body of lambda functions must be constituted of only one expression. We can assign a lambda to a variable, and then call it as if it were a regular function:
```python
square = lambda i : i * i
square(10)
```

In Python functions are treated as data, in the sense that they're also stored in memory at a certain location. To get this information, for example, we could just print a function by name:
```python
def get_msg():
  return  'Hello Python World!'
print(get_msg) # Output: <function get_msg at 0x10ad961e0>
```

Functions, like data, also have a type:
```python
print(type(get_msg)) # Output: <class 'function'>
```

So a function is really an instance of the `Function` class, and the name of the function is a pointer to that instance in memory. We can then assign a function object to a different variable:
```python
get_msg_alias = get_msg
print(get_msg_alias()) # Same as get_msg()
```

and change what an existing function name points to:
```python
def get_some_other_msg():
  return 'Some other message!!!'
get_msg = get_some_other_msg
print(get_msg()) # Output: 'Some other message!!!'
```

Of course this allows us to define higher-order functions, which either take other functions as arguments, or return other functions as results:
```python
def apply(x, function):
  result = function(x)
  return result
```
```python
def fun(n):
  return lambda n: n%2 == 0
```

We can define functions inside of functions, and return them:
```python
def make_function():
  def adder(x, y):
    return x + y

  return adder
```

We can produce a curried function from a regular one:
```python
def multby(a):
  return lambda y: a * y

double = multby(2)
print(double(3)) # output: 6
triple = multpy(3)
print(triple(3)) # output: 9
```

We can refer to a function pointer from within another function using `global`:
```python
def increment(num):
  return num + 1

def reset_function():
  global increment
  increment = lambda num: num + 2
```

## Classes and objects

In Python everything is an object, and thus an instance of a specific class. For example, integers are instances of the `int` class, as described by a call like `type(4)`, returning `<class 'int'>`. We can also use the `isinstance()` function to check if a value is an instance of a certain class:
```python
isinstance(1, int) # evaluates to True
```

To define a new class:
```python
class Person:
  """ An example class to hold a
      persons name and age"""
  def __init__(self, name, age):
    self.name = name
    self.age = age
```

the constructor being the `__init__()` method. Instance methods are all those who take as first parameter the instance variable, named `self`, which carries the state of the instance, like with `self.name` and `self.age`, and the `self` parameter is not filled by the calling code, but by the Python runtime. The documentation is again accessible from the static property `__doc__` like with `Person.__doc__`. To create a new instance of a class:
```python
p1 = Person('John', 36)
p2 = Person('Phoebe', 21)
```

When a new object is created, it is assigned a unique (within the process) identifier by the Python runtime:
```python
id(p1) # evaluates to 4547191808
id(p2) # evaluates to 4547191864
```

this can be helpful for example to tell if two variables point to the same instance:
```python
p1 = Person('John', 36)
px = p1
id(p1) # evaluates to 4547191808
id(px) # evaluates to 4547191808
```

We can also get the actual memory address where an object is stored, by converting the object to string:
```python
str(p1) # evaluates to <__main__.Person object at 0x10f08a400>
```

We can override the method to cast an object to string:
```python
class Person:
  def __init__(self, name, age):
    self.name = name
    self.age = age

  def __str__(self):
    return self.name + ' is ' + str(self.age)
```

We can manually free the memory allocated to store an object:
```python
p1 = Person('John', 36)
del p1
```

alternatively, setting the variable to `None` has the same effect:
```python
p1 = Person('John', 36)
p1 = None
```

Of course, once the variable reaches the end of its scope of definition, it will automatically be deleted by the Python Memory Manager, which is Python's garbage collector.

Classes have the following intrinsic attributes:
- `__name__`: the name of the class
- `__module__`: the module the class belongs to
- `__bases__`: a collection of the superclasses (base classes) of this class
- `__dict__`: a dictionary of the class attributes
- `__doc__`: the documentation string

Class instances have the following intrinsic attributes:
- `__class__`: the name of the class of the object
- `__dict__`: a dictionary of the object attributes

Class fields can be defined outside of the scope of any method:
```python
class Person:
  instance_count = 0
  def __init__(self):
    Person.instance_count += 1
```

Class methods are instead defined by using the `@classmethod` decorator:
```python
class Person:
  @classmethod
  def increment_instance_count(cls):
    cls.instance_count += 1

  def __init__(self):
    Person.increment_instance_count()
```

where the first parameter `cls` is automatically injected by the runtime, and represents the class itself.

Static methods are defined by using the `@staticmethod` decorator:
```python
class Person:
  @staticmethod
  def static_function():
    print('Static method')
```

and they aren't injected with the class itself.

We can define a subclass, extending another class, as:
```python
class Employee(Person):
  def __init__(self, name, age, id):
    super().__init__(name, age)
    self.id = id
```

To refer to methods and properties defined in the parent class we use the `__super()__` method which return a reference to the instance object, but as if its type were the parent class, thus allowing us to access the parent class methods and properties.

If no superclass is specified, the class implicitly extends the `object` class, which means that at the top of any classes hierarchy there is always the `object` class, which provides the following special methods and attributes:
- `__str__()`
- `__init()__`
- `__eq()__`
- `__hash()__`
- `__class__`
- `__dict__`
- `__doc__`
- `__module__`

Methods whose name start with an underscore are treated specially by the Python compiler: in particular they are mangled by prefixing them with the class name, so that a class member called `__somename` will end up being named `_classname__somename` in the bytecode. This has been used to define the convention that members starting with a single underscore should be regarded as *protected* members, while members starting with a double underscore should be regarded as *private* members.

Python supports multiple inheritance:
```python
class ToyCar(Car, Toy):
  """ A Toy Car """
```

and to solve the diamond problem, Python employs a breadth first search to select methods from parent classes. For example, if both parent classes define a `move()` method, when calling `move()` on the subclass, the parent classes are inspected in the order they are declared in the subclass definition: in this example the `Car` super class is first inspected to look for a `move()` definition, and only if none is found there, the `Toy` class is inspected then.

Classes can define overloaded operator methods:
```python
class Quantity:
  def __init__(self, value = 0):
    self.value = value
  def __add__(self, other):
    new_value = self.value + other.value
    return Quantity(new_value)
  def __sub__(self, other):
    new_value = self.value - other.value
    return Quantity(new_value)

q1 = Quantity(5)
q2 = Quantity(10)
q3 = q1 + q2 # evaluates to Quantity(15)
q4 = q2 - q1 # evaluates to Quantity(5)
```

Special methods for numerical operators are:
- `__add__(self, q2)`: `q1 + q2`
- `__sub__(self, q2)`: `q1 - q2`
- `__mul__(self, q2)`: `q1 * q2`
- `__pow__(self q2)`: `q1 ** q2`
- `__truediv__(self, q2)`: `q1 / q2`
- `__floorfiv__(self, q2)`: `q1 // q2`
- `__mod__(self, q2)`: `q1 % q2`
- `__lshift__(self, q2)`: `q1 << q2`
- `__rshift__(self, q2)`: `q1 >> q2`

Since any value can be passed to methods as arguments, we can let our operators work also with other data types:
```python
class Quantity:
  ...
  def __mul__(self, other):
    if isinstance(other, int):
      new_value = self.value * other
    else:
      new_value = self.value * other.value
    return Quantity(new_value)
```

Special methods for comparison operators are:
- `__lt__(q1, q2)`: `q1 < q2`
- `__le__(q1, q2)`: `q1 <= q2`
- `__eq__(q1, q2)`: `q1 == q2`
- `__ne__(q1, q2)`: `q1 != q2`
- `__gt__(q1, q2)`: `q1 > q2`
- `__ge__(q1, q2)`: `q1 >= q2`

Special methods for logical operators are:
- `__and__(q1, q2)`: `q1 & q2`
- `__or__(q1, q2)`: `q1 | q2`
- `__xor__(q1, q2)`: `q1 ^ q2`
- `__invert__()`: `~q1`

Python allows to define actual properties for classes:
```python
class Person:
  def __init__(self, name, age):
    self._name = name
    self._age = age

  @property # getter
  def age(self):
    return self._age

  @age.setter # setter
  def age(self, value):
    if isinstance(value, int) & value > 0 & value < 120:
      self._age = value

  @property
  def name(self):
    return self._name

  @name.deleter # deleter
  def name(self):
    def self._name

p = Person('Bob', 41)
p.age # evaluated to 41
p.age = 42
p.name # evaluated to 'Bob'
del p.name
```

Here we've defined a read-write property `age`, and a read-only property `name`. Of course the `_name` and `_age` fields are still publicly accessible in the class, even though we're using the underscore convention to declare them as private.

## Errors

In Python errors are objects whose types all descend from the `BaseException` class, which has the only subclass `Exception`. Errors are handled by this:
```python
try:
  # code
except ZeroDivisionError as e:
  # handle ZeroDivisionError
except IndexError as e:
  # handle IndexError
except Exception as e:
  # fallback for other types of exceptions
```

## References

- "A Beginners Guide to Python 3 Programming" by John Hunt, Springer 2019
