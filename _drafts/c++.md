# C++

## Program organization

C++ provides by default a global scope where we can define the program's elements: this is what happens if we just define them, adding no other scope specification. However, leaving definitions in the global scope will quickly make the code messy, and introduce issues such as names conflicts.

To address this issue, we can resort to various organizational tools, of which the one at the highest level are *namespaces*, which allow to reuse identifiers multiple times without conflicts. To define a namespace:

```c++
namespace mycode {
  void foo() {
    // ...
  }
}
```

Identifiers defined inside a namespace are accessible from outside of it by using the namespace resolution operator `::`:

```c++
mycode::foo()
```

The `using namespace` directive is useful for avoiding too many resolutions, by letting all identifiers defined in an external namespace to be visible from the current namespace, without the need to resolve them:

```c++
using namespace mycode;
// ...
foo();
```

We can add multiple `using namespace` directives, but we need to be careful because different namespaces might be defining the same identifiers, which would be in conflict with each other.

The `using` declaration allows to declare a single identifier belonging to a different namespace, so that it can be used in the current namespace without needing to be resolved:

```c++
using mycode::foo;
// ...
foo();
```

`using` directives and declarations should never be used in the global scope in a header file, because then whoever will include that header will be forced to deal with the resolutions we used. It's best to add resolutions where they are needed, for example at the beginning of a namespace or class definition.

We can define namespaces inside other namespaces:

```c++
namespace MyLibraries {
  namespace Networking {
    namespace FTP {
      // ...
    }
  }
}
```

Which, starting with C++17, can be rewritten as:

```c++
namespace MyLibraries::Networking::FTP {
  // ...
}
```

We can define *namespace alias* to rename an existing namespace name or resolution:

```c++
namespace MyFTP = MyLibraries::Networking::FTP;
```

Every C++ program must contain one and only one definition of a `main` function that must have this signature:

```c++
int main(int argc, char* argv[])
```

The `main` function is the entry point of the program, the code that the operating system calls when it spawns a new process to run the program. The operating system passes the number of process arguments in the `argc` argument, and their actual values in the `argv` argument. The value of `argv[0]` is platform dependent: in some operating systems it's the program name, in others it's an empty string, but it never contains the first actual argument.

[2]

## Variables

When a variable is declared, the runtime allocates an area of memory to store the value of the new variable; the size of this area is determined by the type of the variable, and the address in memory of this area is chosen by the runtime, among the free memory available. When we access a variable that has just been declared, then, the runtime reads whatever bytes are currently stored in that area of memory, which have been likely written by our application at a different time. Since these bytes, for our purposes, are completely random, it's a good practice to just overwrite them with a sensible initialization value, before we start using the variable.

We can declare new variables using the *uniform initialization* syntax (since C++11):

```c++
int initializedInt { 7 };
```

Alternatively, we can use the traditional initialization syntax:

```c++
int initializedInt = 7;
```

[2]

## Primitive types

C++ provides several primitive types out of the box. The most simple of them are numeric and bytes types:

- `int`: positive and negative integers of average size (usually 4 bytes); for example `int i { -7 }`
- `short`: like `int`, but smaller (usually 2 bytes); for example `short s { 13 }`
- `long`: like `int`, but potentially bigger (in actuality usually still 4 bytes); for example `long l { -7L }`
- `long long`: like `int`, but potentially bigger (at least the same size of `long`, but usually 8 bytes); for example `long long ll { 14LL }`

Integers declarations might be enhanced with the `signed` or `unsigned` qualifiers, like `signed int` or `unsigned long`; the `signed` qualifier is not necessary because by default integers are declared as signed, so `signed int` is the same as `int`, but they can be useful for expressiveness. Integers types different than `int` might still be qualified with the `int` type to remember that they are integers, like with `short int`, `long int` etc. This is true because `int` is treated as the default type, and thus can be omitted if another qualifier is present: this also means that we can write `signed` in place of `signed int` and `unsigned` in place of `unsigned int`.

Values that need to be stored in a non-`int` variable should be qualified with the actual type they'll be stored as.

[2]

## Custom types

Unlike C, in C++ it's not necessary to use `typedef` with `struct` in order to simplify the definition of the identifier: this means that in C++ we can just define a `struct` with:

```c++
struct student {
  // ...
};
```

and then use it with:

```c++
student s1;
```

[3]

## Software architecture

Software architecture is defined as the set of software elements that describe how a software system should work, along with their relations and properties. According to the kind of system we are considering, and its complexity, we can define different types of software architecture:

- *enterprise architecture* describes how the business of a whole company is related to its IT
- *solution architecture* describes how one specific system is defined, and how it interacts with its surroundings
- *software architecture* describes how one specific project should be realized, with what technology and how it interacts with other projects
- *infrastructure architecture* describes the infrastructure a software will use, including deployment environments and strategy, scalability techniques, failover handling, etc.

Architecture should be thought of in advance, because it's very hard to change architectural decisions after a good part of a system has already be built. Additionally, we need to constantly monitor that every design decision adheres to the agreed architectural choices. Failing to achieve these goals will lead to producing software components that end up being more and more coupled with time.

In an agile development setting, we're not going to take all the architectural decisions of the project in advance, because our development strategy should be able to embrace change: instead, we're going to take small architectural choices upfront, that are enough to enable the current development iteration. By keeping track of these decisions and their rationale, we are able to update them in future iterations, when the need comes, thus adopting the so-called *evolutionary architecture*.

Architectural choices are driven by the system *requirements*, which can be split into functional requirements, quality attributes and constraints.

*Functional requirements* describe what the software system should do, from a business point of view, i.e. the functionality that it should provide to its users. On the other hand, non-functional requirements are related to various aspects of the software system, that don't impact its ability to deliver the required functionality. Non-functional requirements come in two flavours: *quality attributes* and *constraints*. Quality attributes are the quality traits that the system should have, related to how "well" the system should be able to work, like usability, accessibility, performance, etc. Constraints are non-negotiable decisions that must be followed by the project development, like time, budget, design, technology, etc.

Not all requirements impact the architecture of the system, though: those that do are called *architecturally significant requirements, ASRs*. Requirements that are not architecturally significant can be implemented regardless of the chosen architecture decisions. For example, consider an application that should allow users to filter products from a list: depending on the scale of the app, we might need at some point to add a dedicated search engine, like Lucene or ElasticSearch, which would definitely be impacting the architecture of the system. Another example would be a requirement asking to send notifications to subscribed users, which would definitely need to add a service to handle messages and events.

There are several ways to recognize an architecturally significant requirement:

- if we need to integrate with an external system, like for sending emails, pushing notifications, etc.
- if a requirement has a significant impact on the system, like core functionality or cross-cutting concerns like authorization or auditability
- if a requirement is hard to achieve, like low latency
- if a requirement limits the product, or add constraints in any way
- if the system is part of a specific environment, like a micro-controller versus the cloud, or a team of experienced developers versus one of juniors

An interesting way for documenting requirements is the *Easy Approach to Requirements Syntax (EARS)*, which allows the following statements only:

- Ubiquitous requirement: "The `$SYSTEM` shall `$REQUIREMENT`", for example: "The application will be developed in C++"
- Event-driven: "When `$TRIGGER` `$OPTIONAL_PRECONDITION` the `$SYSTEM` shall `$REQUIREMENT`, for example: "When an order arrives, the gateway will produce a `NewOrderEvent`"
- Unwanted behavior: "If `$CONDITION`, then the `$SYSTEM` shall `$REQUIREMENT`", for example: "If the processing of the request takes longer than 1 second, the tool will display a progress bar"
- State driven: "While `$STATE`, the `$SYSTEM` shall `$REQUIREMENT`", for example: "While a ride is taking place, the app will display a map to help the driver navigate to the destination"
- Optional feature: "Where `$FEATURE`, the `$SYSTEM` shall `$REQUIREMENT`", for example: "Where A/C is present, the app will let the user set the temperature through the mobile application"

These can then be combined to produce more complex requirements, like: "When using a dual-server setup, if the backup server doesn't hear from the primary one for 5 seconds, it should try to register itself as a new primary server".

[1]

## References

1. "Software Architecture with C++", Ostrowski, Gaczkowski, 2021
2. "Professional C++ Fifth Edition", Gregoire, 2021
3. "Demystified Object-Oriented Programming with C++", Kirk, 2021
