## Idiomatic Python

The Python community has agreed on a common naming and formatting standard, which is described in [PEP8](http://www.python.org/dev/peps/pep-0008/).

Tuples can be used in Python to function as a destructuring operator, i.e. to assign multiple variables from the values of a list on a single line. So instead of writing:
```python
list_from_comma_separated_value_file = ['dog', 'Fido', 10]
animal = list_from_comma_separated_value_file[0]
name = list_from_comma_separated_value_file[1]
age = list_from_comma_separated_value_file[2]
```

we should write:
```python
list_from_comma_separated_value_file = ['dog', 'Fido', 10]
(animal, name, age) = list_from_comma_separated_value_file
```

One simple application of the tuple destructuring is to swap variables:
```python
(foo, bar) = (bar, foo)
```

Comparison operators can be chained arbitrarily, and that's usually better for readability:
```python
3 <= 2 # False
3 <= 3 <= 4 # True
3 <= 2 <= 4 <= 4 # False
1 <= 2 <= 3 <= 2 # True
1 <= 2 >= 1 # True
```

While it's possibile in Python to add more statements to the same line, by separating them with a semicolon, it's more idiomatic to put every statement in its own line, with proper indenting. So instead of:
```python
name = 'John'; address = 'Kampala'
if name: print(name)
print(address)
```

it's preferable to write:
```python
name = 'John'
address = 'Kampala'

if name:
    print(name)

print(address)
```

Instead of explicitly comparing to `True`, `False` or `None`, it's preferable to leverage the automatic conversions to boolean. "falsy" values are those that would be converted to `False` inside a conditional expression, like empty lists `[]`, empty dicts `{}`, `None`, `False` and `0`. Almost everything else is instead considered "truthy", i.e. would be converted to `True`.
```python
(x, y) = (True, 0)

if x:
    # x is "truthy"
else:
    # x is "falsy"

if not y:
    # y is "falsy"
```

Short conditional blocks can be inlined, and treated as expressions, i.e. returning a value: this would actually replace the ternary operator that is not available in Python:
```python
a = True
value = 1 if a else 0
```

The `in` operator can be used to test whether a value is included in a collection, and can be used to replace a complex chain of `or` conditionals. So instead of writing:
```python
city = 'Nairobi'
found = False

if city == 'Nairobi' or city == 'Kampala' or city = 'Lagos':
    found = True
```

write:
```python
city = 'Nairobi'
found = city in ['Nairobi', 'Kampala', 'Lagos']
```

The `in` operator also works inside loops, so instead of writing:
```python
cities = ['Nairobi', 'Kampala', 'Lagos']
index = 0

while index < len(cities):
    print(cities[index])
    index += 1
```

write:
```python
cities = ['Nairobi', 'Kampala', 'Lagos']
for city in cities:
    print(city)
```

Prefer using multiple assignment when possible:
```python
x = y = z = 'Foo'
```

Refrain from using the `+` operator to concatenate strings, since it's slow, consumes more memory, and it's harder to read. To just concatenate string, we can use the `join` method:
```python
result_list = ['True', 'False', 'File not found']
result_string = ''.join(result_list)
```

Instead, to interpolate strings with values we can use the `format` method:
```python
def user_info(user):
    return 'Name: {user.name} Age: {user.age}'.format(user=user)
```

When we want to define a set of values based on some conditions, the most verbose way to do it would be to define a first set of elements, and then write a loop to filter only the ones we want:
```python
ls = list()

for element in range(10):
    if not(element % 2):
        ls.append(element)
```

A more functional approach would be to use the `filter` function over the original collection, with a lambda function:
```python
ls = list(filter(lambda element: not(element % 2), range(10)))
```

However, the more idiomatic way in Python would be to use list comprehension, that allows to define a list directly its properties:
```python
ls = [element for element in range(10) if not (element % 2)]
```

Comprehension works also for dictionaries, so instead of writing:
```python
emails = {}

for user in users:
    if user.email:
        emails[user.name] = user.email
```

we should write:
```python
emails = {user.name: user.email for user in users if user.email}
```

Comprehension also works for sets, so instead of writing:
```python
elements = [1, 3, 5, 2, 3, 7, 9, 2, 7]
unique_elements = set()

for element in elements:
    unique_elements.add(element)
print(unique_elements)
```

we can write:
```python
elements = [1, 3, 5, 2, 3, 7, 9, 2, 7]
unique_elements = {element for element in elements}
print(unique_elements)
```

The `enumerate` function turns a list into a list of tuples containing the index of each element of the list, along with the element itself, and it's idiomatically used when we want to iterate over a list having also access to the indexes. So instead of writing:
```python
ls = list(range(10))
index = 0

while index < len(ls):
    print(ls[index], index)
    index += 1
```

we should write:
```python
ls = list(range(10))
for index, value in enumerate(ls):
    print(value, index)
```

Sets can be used to simplify the task of creating lists out of other lists. For example, if we want to calculate the intersection of two lists, instead of writing complex loops:
```python
ls1 = [1, 2, 3, 4, 5]
ls2 = [4, 5, 6, 7, 8]

elements_in_both = []
for element in ls1:
    if element in ls2:
        elements_in_both.append(element)
print(elements_in_both)
```

we can write:
```python
ls1 = [1, 2, 3, 4, 5]
ls2 = [4, 5, 6, 7, 8]

elements_in_both = list(set(ls1) & set(ls2))
print(elements_in_both)
```

Python provides a clean way to employ the RAII principle to ensure resources are always properly disposed of. The problem we want to avoid is that an exception is thrown before the resource has had the chance to be disposed, like it happens in this case:
```python
file_handle = open(path_to_file, 'r')
for line in file_handle.readlines():
    if some_function_that_throws_exceptions(line):
        # do something
file_handle.close()
```

Instead, we can make sure that the resource is closed no matter what:
```python
with open(path_to_file, 'r') as file_handle:
    for line in file_handle:
        if some_function_that_throws_exceptions(line):
            # do something
```

here instead of manually creating a file handle, that requires explicit calls to `readlines` and `close`, the `with` block creates a "file context manager" that already assumes we want to get the lines from the file, and automatically catches exceptions from the given code, and closes the file before rethrowing them. There are various types of context managers already available for different kinds of resources, but it's also possible to write customized ones.

A very useful library that should almost always be taken into account is [itertools](https://docs.python.org/3/library/itertools.html#recipes).

## References
- https://medium.com/the-andela-way/idiomatic-python-coding-the-smart-way-cc560fa5f1d6
- https://www.jeffknupp.com/blog/2012/10/04/writing-idiomatic-python/