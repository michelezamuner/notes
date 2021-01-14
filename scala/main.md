"Scala for Java Developers", Weston, 2018

## Program entry point

Every Scala program must provide an object defining a method having the signature `main(Array[String])`, which will be used by the virtual machine as the entry point. Unlike Java, in Scala we can define literal singleton objects in addition to classes:
```scala
object Main {
    def main(args: Array[String]) {
        println("Hello, " + args.toList)
    }
}
```

The name of the object carrying the `main` method is irrelevant; however, it must be set because literal objects without a name, and not being used right away in the context of their definition, wouldn't be accessible.

## Expressions

In Scala expressions may not end with a semicolon, because the compiler can understand that the expression terminates with the new line. If we want to write multiple expressions on the same line, however, we need to separate them with a semicolon.

## Values and variables

Data identifiers in Scala can be "values" or "variables". Values are constant, meaning that they will always point to the same data instance:
```scala
val language = "Scala"
language = "Java" // error: reassignment to val
```
```scala
val obj = new Object { var language = "Scala"}
obj.language = "Java" // no error
```

here in the second example the value `obj` always points to the same object, even if that object is changed later on. Thus, `val` can be referenced to as a constant reference to an object, and not as a reference to a constant object, because the objects it can point to can indeed be mutable.

Variables are instead mutable references to objects, meaning that they can point to different objects in the course of their lifetime:
```scala
var language = "Scala"
language = "Java" // no error
```

Data identifiers don't need to be decorated with a type because Scala can infer it from the value that is passed in the assignment. However, we can also explicitly declare the identifier type after its name:
```scala
val language: String = "Scala"
```

## Functions

Functions can be defined with the `def` keyword. Types must be present in the signature because otherwise there wouldn't be any other way for the compiler to know them:
```scala
def min(x: Int, y: Int): Int = {
    if (x < y)
        return x
    else
        return y
}
```

In Scala the last statement of a function can also be automatically used as the return value, so the previous function can be rewritten as:
```scala
def min(x: Int, y: Int): Int = {
    if (x < y)
        x
    else
        y
}
```

where, since the first `x` is already returning, can be further simplified to:
```scala
def min(x: Int, y: Int): Int = {
    if (x < y)
        x
    y
}
```

Unlike the parameters' types, the return type can be inferred by the compiler by looking at the actual function code:
```scala
def min(x: Int, y: Int) = {
    if (x < y) x else y
}
```

here we omitted the return type declaration because from the code it's clear that it will be an `Int`. We also wrote the conditional in one line, for doing which we had to add back the `else`. We can further simplify this as:
```scala
def min(x: Int, y: Int) = if (x < y) x else y
```

which clearly shows how in Scala a function definition is just a special kind of assignment, where we assign a code block to an identifier, which is the function signature.

If a function returns nothing, the equals sign at the end of the signature must be omitted. The equals sign tells the compiler that something must be returned, and that's why we can omit the `return` keyword in the code.
```scala
def greet(msg: String) {
    println(msg)
}
```
```scala
def min(x: Int, y: Int) {
    if (x < y) x else y // warning: a pure expression does nothing in statement position
}
```

in the second example here we see how, by not declaring that the function will return a value, the `x` and `y` that are left alone with no `return` are considered like normal statements that should produce side effects, but since they don't, the compiler raises a warning, thinking that the programmer has forgot to add something.

If no equals sign is used in the definition of the function, the implicit return type that is assumed is the special type `Unit`. If we try to return something else, that will be ignored:
```scala
def min(x: Int, y: Int) {
    if (x < y)
        return x // warning: enclosing method min has result type Unit: return value of type Int discarded
    return y
}
```

additionally, the compiler will still try to regard that `x` as an expression with a side effect, raising a warning because it is not:
```scala
def min(x: Int, y: Int) {
    if (x < y)
        return x // warning: a pure expression does nothing in statement position
    return y
}
```

## Operators and method call notation

In Scala operators are normal functions, which are only usually used with infix notation:
```scala
val age: Int = 10
age * .5
```

this is precisely a call to the `*` method of the `Int` class:
```scala
def *(x: Double): Double
```

meaning that we can call it also with the dot notation:
```scala
age.*(.5)
```

Additionally, in Scala literals can also be called methods on, so this also works:
```scala
5.*(10)
```

And on the other hand we can use the infix notation also with methods that are not operators:
```scala
35 toString
```

here we treat `toString` as a unary operator, meaning that we don't use the dot (`.`), nor the parenthesis after the call (`()`).

Infix notation, however, cannot work for methods that have more than one parameter, because there's no way to place a single operator (or method) between three or more arguments. We can still drop the dot, however, with more than one argument:
```scala
"aBCDEFG" replace("a", "A")
```

This means that operators can actually be overloaded, because one can just define a "plus" method, for example, and call it with infix notation:
```scala
train + passenger
```

## Tuples

Tuples in Scala are defined like:
```scala
val tuple = ("save", 50, true)
```

and values are accessed like:
```scala
val event = tuple._1    // "save"
val millis = tuple._2   // 50
val success = tuple._3  // true
```
```scala
val (event, millis, success) = tuple
```

The type of a tuple is the list of types of its values:
```scala
def audit(event: (String, Int, Boolean)) = {
    ...
}
```

where the `event` parameter takes a tuple containing a `String`, a `Int` and a `Boolean`.

## Collections

Collections like `List`, `Set` and `Map` are immutable by default:
```scala
val map = Map(1 -> "a", 2 -> "b")
```

meaning that there is no actual method on `Map` and the others to change their statuses. There are other collection types, however, that are mutable.

To iterate over a list:
```scala
list.foreach(println)
```
```scala
for (value <- list) println(value)
```

## Scala classes

Scala provides a predefined hierarchy of classes from the `scala` package. At the top of the hierarchy stands the `Any` class, from which every other Scala class derives. `Any` provides an interface with some common methods shared by all Scala classes:
```scala
abstract class Any {
    final def ==(that: Any): Boolean
    final def !=(that: Any): Boolean
    def equals(that: Any): Boolean
    def ##: Int
    def hashCode: Int
    def toString: String
    // ...
}
```

Scala provides both `equals` and `==` to represent natural equality, meaning that both methods wrap the `equals` method of Java, thus not comparing if two objects are the same instance, but if they have the same value. For example `42 == 42` in Scala equals to `new Integer(42).equals(new Integer(42))` in Java. There are two methods doing the same thing because `equals` can be overriden by derived classes to do something else, while always keeping the original equality available with `==` which in fact is `final` and thus will never behave differently than its original definition.

`Any` has two subclasses, which are `AnyVal` and `AnyRef`, defining the so-called *value types* and *reference types*. Value types are all predefined, meaning that a user cannot define new value types; they correspond to Java's stack-allcated primitive types, even though in Scala they're implemented as objects as well, and they're allocated on the heap; all value types can be referenced to by literals in the code. Value types in Scala are the following: `Byte`, `Short`, `Char`, `Int`, `Long`, `Float`, `Double`, `Boolean`, `Unit`. For value types there is no identity equality method defined, because for value types it makes no sense to worry if two objects are the same instance or not, since they're not designed to have an identity.

In Scala the `Unit` type represents a missing value, much like Java's `Void`. It can be represented by the literal `()`, thus allowing initializations like `val example: Unit = ()`, or methods returning nothing like `def call: Unit = ()`, where `call()` would return the value of the last expression, which in this case is still `()`.

Reference types are used to represent any Scala object expect for those represented by the value types, and they map to Java's `java.lang.Object`: for example Scala's `String` class extends `AnyRef`. Unlike value types which all have different implementations of base methods like `toString`, `equals` and `hashCode`, Scala provides default implementations for these methods for all reference types. Since it makes sense for reference types to have an identity, `AnyRef` additionally provides the `eq` and `ne` methods, which check if two objects are in fact the same instance, regardless of their natural equality, so that for example `new String("A") eq new String("A")` would be `false`.

The Scala class hierarchy also provides the so-called *bottom types*, which are types that are subtypes of all types, like `Null` and `Nothing`. `Null` represents the absence of an object reference, and as such is subclass of `AnyRef`; additionally, all reference types are super types of `Null`, in order for `Null` to be allowed to be used in place of any reference type. On the other hand, `Nothing` is subtype of both `AnyVal` and `AnyRef`, and thus it's also subtype of `Null`; all Scala types are hence super-types of `Nothing`.

A custom class in Scala can be as simple as:
```scala
class Counter
```

We can see what would be the equivalent Java code for the same JVM bytecode generated by this Scala code by following this procedure:
1. compile this Scala code with:
```
$ scalac counter.scala
```
2. the previous step would generate a `Counter.class` file; we can then use the Jad utility to generate the equivalent Java code:
```
$ jad Counter
```
3. the previous step would generate a `Counter.jad` file, which we can inspect to see the actual Java code

The previous Scala code would be equivalent to the following Java code:
```java
public class Counter {
    public Counter() {
    }
}
```

A public constant field can be defined just with a `val` declaration:
```scala
class Counter {
    val count = 0
}
```

which would translate to:
```java
public class Counter {
    private final int count = 0;
    public Counter() {
    }
    public int count() {
        return count;
    }
}
```

So we see that Scala automatically creates a public getter for the constant value `count` that has been defined. However, in Scala this getter cannot be invoked with a method call:
```scala
val counter = new Counter
println(counter.count()) // error: Int does not take parameters
```

because it has not been defined in the Scala code as a method taking arguments, even though it's Java representation will actually be a real method.

To make this field private, we need to explicitly declare it as such:
```scala
class Counter {
    private val count = 0
}
```
```java
public class Counter {
    private final int x = 1;
    public Test() {
    }
    private int x() {
        return x;
    }
}
```

where the getter is still defined, even if it'll be accessible only form inside the class anyway.

For variable fields, Scala provides a setter as well:
```scala
class Counter {
    private var count = 0
}
```
```java
public class Counter {
    private int count;
    public Test() {
        count = 0;
    }
    private int count() {
        return count;
    }
    private void count_$eq(int count$1) {
        count = count$1;
    }
}
```

First of all, we see that the default value of the field is set in the Java's class constructor. Additionally, no traditional setter like `setCount` is defined, and this is because Scala prefers to use an assignment-like syntax to set values to fields:
```scala
class Counter {
    private var count = 0

    def increment() {
        count += 1
    }
}
```

where of course `count += 1` is equivalent to the assignment `count = count + 1`, which in fact results into the invocation of the `count_$eq` setter method above. The reason why `count_$eq` is used as method name is that the JVM doesn't allow methods to contain the `=` character, so `$eq` is used in its place. Unlike the getter case, here we also have the possibility of directly invoking the setter method:
```scala
def increment() {
    count_$eq(1)
}
```
```scala
def increment() {
    count_=(1)
}
```

and from the second example we understand that this has to work because this is how Scala handles operators definition and infix notation.

In Scala the constructor definition can be replaced with a list of fields right in the class signature:
```scala
class Customer(val name: String, val address: String)
val eric = new Customer("Eric", "29 Acacia Road")
println(eric.name)
```
```java
public class Customer {
    private final String name;
    private final String address;
    public Customer(String name, String address) {
        this.name = name;
        this.address = address;
    }
    public String name() {
        return this.name;
    }
    public String address() {
        return this.address;
    }
}
```

The properties thus defined are also already public, and read-only since they've been defined as `val`.

Any code that is found in the body of the class definition will end up as part of the primary constructor, so the entire class definition can be seen as one big method, equivalent to the primary constructor, since we can define also constructor arguments after the class name:
```scala
class Customer(forename: String, initial: String, surname: String) {
    val fullname = String.format("%s %s. %s", forename, initial, surname)
}
```
```java
public class Customer {
    ...
    private final String fullname;
    public Customer(String forename, String initial, String surname) {
        ...
        this.fullname = String.format("%s %s.%s", forename, initial, surname);
    }
    ...
}
```

We can also define auxiliary constructors, which go by the method name `this`:
```scala
class Customer(forename: String, initial: String, surname: String) {
    val fullname = String.format("%s %s.%s", forename, initial, surname)

    def this(name: NameType) {
        this(name.forename, name.initial, name.surname)
    }
}
```

auxiliary constructors must always include a call to another constructor, again using `this()`, in order to make sure that the primary constructor is always called in the end, and this call must always be the first line of the auxiliary constructor body, otherwise the code won't compile.

In Java we need to define multiple constructors in order to handle default values, like:
```java
public class Customer {
    private final String fullname;
    public Customer(String forename, String initial, String surname) {
        this.fullname = String.format("%s %s. %s", forename, initial, surname);
    }
    public Customer(String forename, String surname) {
        this(forename, "", surname);
    }
}
```

Scala methods, on the other hand, support default values, so in this case we can just use one constructor definition:
```scala
class Customer(forename: String, initial: String = "", surname: String) {
    val fullname = String.format("%s %s. %s", forename, initial, surname)
}
```

Of course we can use named arguments to make it easier to use default values:
```scala
new Customer("Bob", surname = "Smith")
```
instead of:
```scala
new Customer("Bob", "", "Smith")
```

Scala provides the special `object` syntax to create singletons:
```scala
object Logger {
    def log(level: Level, string: String) {
        printf("%s %s\n", level, string)
    }
}
```

the `Logger` identifier would already refer to an object, and not to a class. In actuality, no class is being defined, and no new instance can be created with `new`.

Singleton objects can be defined with the same name as existing classes in the same namespace: these would be called "companion objects":
```scala
class Customer(val name: String, val address: String) {
    private val id = Customer.nextId()
}

object Customer {
    private var lastId = 0

    private def nextId(): Integer = {
        lastId += 1
        lastId
    }
}
```

Companion objects can also access private fields of the classes with the same name. Companion objects are mainly used to simulate static methods (like factories), which are not natively supported in Scala, for example when we want to provide namespaced raw functions.
```scala
class Customer(val name: String, val address: String)

object Customer {
    def apply(name: String, address: String) = new Customer(name, address)
}
```

this would be used like:
```scala
Customer.apply("Bob Fossil", "1 London Road")
```

or just:
```scala
Customer("Bob Fossil", "1 London Road")
```

this because in Scala the special `apply` method is used to define callable objects.

Once we have static constructors, we can even make the primary constructor private to force the way the objects will be created:
```scala
class Customer private (val name: String, val address: String)
```
```java
public class Customer {
    private final String name;
    private final String address;
    public static Customer createCustomer(String name, String address) {
        return new Customer(name, address);
    }
    private Customer(String name, String address) {
        this.name = name;
        this.address = address;
    }
}
```

## Higher-order functions

An anonymous, or lambda, function, is defined like this:
```scala
(a: String, b: String) => {
    val comparison = a.compareTo(b)
    comparison
}
```

if arguments types can be inferred, this can be abbreviated to:
```scala
(a, b) => a.compareTo(b)
```

if there's only one argument, parenthesis can be omitted:
```scala
dollar => dollar * 0.76
```

if there's no argument, a couple of empty parenthesis must be used instead:
```scala
() => println("Hello, World!")
```

Lambdas are implemented in Scala as instances of single method interface (SAM), which is one of the `java.util.function` interfaces, like `java.util.function.Function` for single argument functions or `java.util.function.BiFunction` for two arguments functions. In Scala we instead use `scala.Function2[T1, T2, T3]` to represent a function taking two arguments and returning a value, for example `scala.Function2[Int, Int, Int]` represents a function taking two integers and returning another one. We can use this type to store a function into a variable:
```scala
val plus: Function2[Int, Int, Int] = (a: Int, b: Int) => a + b
```

However, in Scala we can use the *function type* instead of the function class above:
```scala
val plus: (Int, Int) => Int = (a: Int, b: Int) => a + b
```

which can further be simplified by removing the argument types from the declaration, since they're already included in the function type:
```scala
val plus: (Int, Int) => Int = (a, b) => a + b
```

Instances of function types always provide the `apply` method to allow them to be called in an object oriented way, so we can write:
```scala
plus.apply(2, 2)
```

even though it's usually better to just invoke the object as a function:
```scala
plus(2, 2)
```

If a method takes as a parameter a type that is not a `scala.Function` nor a function type, we can still implicitly coerce a lambda into that type. Let's say we want to use the Java `sorted()` method, belonging to lists, which takes a `Ordering<T>` as parameter:
```scala
def sort: List[Customer] = {
    items.toList.sorted(new Ordering[Customer] {
        def compare(a: Customer, b: Customer) = b.total.compare(a.total)
    })
}
```

it would be better if we could pass a lambda instead of an anonymous class instead, since the `Ordering` type defines only the `compare` methods (and it's thus a functional interface). To do this, we can create an `implicit` generic function that creates an `Ordering` of any type:
```scala
implicit def functionToOrdering[A](f: (A, A) => Int): Ordering[A] = {
    new Ordering[A] {
        def compare(a: A, b: A) = f.apply(a, b)
    }
}
```
then we can pass a lambda to our `sorted` call:
```scala
items.toList.sorted((a: Customer, b: Customer) => b.total.compare(a.total))
```

This works because once `functionToOrdering` is registered as `implicit`, Scala knows that every time an `Ordering` is expected, but a function with the correct signature is provided instead, it should use that implicit function to produce the required `Ordering`. Actually, the name of the function, `functionToOrdering` is quite irrelevant since we are never going to call it, being its usage implicit.

## Inheritance

Like with Java, we can define a subclass using the `extends` keyword, and prevent subclassing using `final`. However, in Scala class definitions are equivalent to primary constructor definitions, so in the subclass we have to repeat the primary constructor arguments:
```scala
class Customer(name: String, address: String)
class DiscountedCustomer(name: String, address: String) extends Customer
```

to call the superclass' constructor, we pass the arguments in the extended class:
```scala
class DiscountedCustomer(name: String, address: String) extends Customer(name, address)
```

to override a method from the parent's class, we have to use the `override` keyword (replacing the `@Override` annotation of Java):
```scala
class Customer(name: String, address: String) {
    def total: Double = {
        // return total
    }
}
class DiscountedCustomer(name: String, address: String) extends Customer(name, address) {
    override def total: Double = {
        super.total * 0.90
    }
}
```

The `override` keyword is not mandatory when overriding fields and abstract methods.

We can specify that a class should not be instantiated, by making it `abstract`:
```scala
abstract class AbstractCustomer {
    def total: Double
}
```

and again in subclasses it's not required to use `override` for the abstract methods.

### Traits

Scala supports traits (or mixins) in substitution of Java's interfaces. Traits can define abstract or concrete methods, and fields, and a class might implement any number of traits, mixing in their behaviors:
```scala
trait Readable {
    def read(buffer: CharBuffer): Int
}
```

methods that are not implemented are automatically abstract, so we don't need to use `abstract` on them. To use this new trait:
```scala
class FileReader(file: File) extends Readable {
    override def read(buffer: CharBuffer): Int = {
        val linesRead: Int = 0
        return linesRead
    }
}
```

To use additional traits, we use the `with` keyword:
```scala
class FileReader(file: File) extends Readable with AutoCloseable {
    def read(buffer: CharBuffer): Int = {
        ...
    }

    def close() {
        ...
    }
}
```

Fields defined in traits can be `protected`:
```scala
trait Counter {
    protected var count = 0
    def increment()
}
```

meaning that they will be visible only to classes using that trait. The default value supplied in the declaration will be assigned to each specific instance's field on construction. Fields can also be "abstract", meaning that no default value is supplied:
```scala
trait Counter {
    protected var count: Int
    def increment()
}
```

and in this case the class using the trait will be forced to provide a value for that field:
```scala
class IncrementByOne extends Counter {
    override var count: Int = 0
    override def increment(): Unit = count += 1
}
```

In case methods with the same signature are used by more than one trait that is used by the same class, Scala has to figure out a way to choose which implementation to use when that signature is invoked. What Scala actually does is taking into account the order with which traits are added to the class definition, and the parents of each trait, in order to sort them by priority. For example if we have:
```scala
class Animal
trait HasWings extends Animal
trait Bird extends HasWings
trait HasFourLegs extends Animal
class FlyingHorse extends Animal with Bird with HasFourLegs
```
```
         Animal
           /\
          /  \
HasFourLegs  HasWings
     |          |
     \          |
      \        Bird
       \        /
        \      /
      Flying Horse
```

and there's a `move` methods on `HasFourLegs` that implements an horizontal movement, while there's a `move` method on `HasWings` that implements a vertical movement, which implementation should be used when we call `move` on a `Flying Horse`? What should happen if it calls `super.move`?

What Scala does in these cases is a process of linearization, flattening the hierarchy from right to left, so that in the previous example this would be produced:
```
Animal <-- HasWings <-- Bird <-- HasFourLegs <-- FlyingHorse
```

if instead the definition was:
```scala
class FlyingHorse extends Animal with HasFourLegs with Bird
```

then the hierarchy would be:
```
Animal <-- HasFourLegs <-- HasWings <-- Bird <-- FlyingHorse
```

## Control structures

In Scala the `if/else` construct is actually an expression, i.e. it returns the value of the chosen branch, which can be then assigned to a variable or used in another way. Unlike Java, then, the preferred style for conditionals is this inline:
```scala
val tall = if (height > 190) "tall" else "not tall"
```

This also means that the ternary operator `?:` has no reason to exist in Scala.

Instead of `switch` 


## References
- https://stackoverflow.com/questions/42320394/scala-string-toint-int-does-not-take-parameters
- https://alvinalexander.com/scala/how-to-disassemble-decompile-scala-source-code-javap-scalac-jad/