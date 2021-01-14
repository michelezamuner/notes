# TypeScript

## Reference

"TypeScript Quickly", Fain and Moiseev, Manning 2020

## Using TypeScript

TypeScript is a superset of the JavaScript programming language, that is compiled down to any specific version of JavaScript to be run on the browser or on Node.js. More precisely, TypeScript supports at any time all the features defined for the ECMAScript language, in addition to TypeScript's own features.

The main purpose of TypeScript is to add optional static analysis to JavaScript. Here, "optional" means that existing non-typed JavaScript code can still be accepted by TypeScript compiled, and that we can mix TypeScript and JavaScript in the same application.

Since TypeScript has been designed from the beginning to be compiled to JavaScript, TypeScript developers had the chance to implement not only all currently existing JavaScript features, but also most future ones, which had been defined by the ECMAScript standard, but not supported on browsers already.

To install the TypeScript compiler locally, run:

```bash
$ npm install -g typescript
```

If we have a TypeScript source file, like `main.ts`, we can compile it by `cd`'ing to that directory, and invoking:

```bash
$ tsc main
```

If there are compilation errors, the compiler will display them in the compilation output, but an object file `main.js` will still be produced. We can prevent compilation from producing output files in case of error with the `--noEmitOnError` compilation option:

```bash
$ tsc --noEmitOnError true main
```

With this call, only when no error is found during the compilation, the output `main.js` file will be produced.

To select the version of JavaScript to target, we can use the `--t` option:

```bash
$ tsc --t ES5 mains
```

The `strictNullCheck` option will prevent `null` to be assigned to variables with a specified type that doesn't include `null`.

The `strictPropertyInitialization` option will prevent class properties from being declared without being initialized.

All options can be specified in a `tsconfig.json` file; invoking just `$ tsc` will then pick up options from that file:

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "outDir": "./dist",
    "noEmitOnError": true,
    "target": "es5"
  }
}
```

The `watch` option will instruct `tsc` to keep running instead of terminating after the first compilation: while running, it will watch for all configured source files, and automatically compile them as soon as it sees them changing.

A new `tsconfig.json` file can be produced by running `tsc --init` on a new project's directory: the `tsconfig.json` file thus produced will have several default values commented out.

## Types

If we don't explicitly specify any type in a variable's declaration, TypeScript will automatically assign the type of the initializer value to it:

```typescript
let taxCode = 1
```

here, the variable `taxCode` will be automatically assigned to type `number`. At this point, if we try to assign this variable a value of a different type, we'll get an error:

```typescript
let taxCode = 1
taxCode = 'lowIncome' // Type 'string' is not assignable to type 'number'.
```

When a type is automatically assigned by the compiler, it's said it's an *inferred* type.

We can add a type annotation to a variable during its initialization:

```typescript
let firstName: string;
let age: number;
```

The type annotations provided by TypeScript are the following:

- `string`: for textual data
- `boolean`: for `true`/`false` data
- `number`: for numeric values
- `symbol`: for symbols, instances of the `Symbol` class
- `any`: to allow any type, i.e. ignoring the type
- `unknown`: like `any`, but no operation can be performed on an `unknown` variable
- `never`: for unreachable code
- `void`: for no value

Like JavaScript, TypeScript also supports the value `null` and `undefined`.

Union types represent the situation where a variable can be of one of multiple possible types:

```typescript
function getName(): string|null {

}
```

here the `getName` function might return a `string` or `null`.

The `never` type can be useful for example for functions that are designed to never return, like infinite loops that power servers.

```typescript
const logger: () => never = () => {
  while (true) {
    console.log("The server is up and running");
  }
}
```

This is different from the `void` type, which assures that the function in fact ends, while producing no output.

Explicitly specifying types during variable initialization is considered redundant, because the type is clear from the initialization value:

```typescript
let name: string = 'John Smith';
```

It's considered better style to avoid adding the type annotation in such cases:

```typescript
let name = 'John Smith';
```

TypeScript allows to specify actual values as types:

```typescript
let name: 'John Smith';
```

meaning that the variable `name` will only accept the string `John Smith` as a value. These can be useful in unions and enums.

If a variable is not initialized, or it's initialized with `null` or `undefined`, TypeScript will apply *type widening* by assigning the `any` type to it. By contrast, *type narrowing* is the practice of using `typeof` or `instanceof` on variables to determine their types, and run different code accordingly.

The `type` keyword allows us to create custom types or type aliases:

```typescript
type Patient = {
  name: string;
  height: Foot;
  weight?: Pound;
}
```

Here the third property, `weight` is defined as optional, with the question mark `?`, meaning that we can avoid assigning a value to it when creating a new value of type `Patient`:

```typescript
let patient: Patient = {
  name: 'Joe Smith',
  height: 5
}
```

If a property is not declared as optional, it means it's required, and the compiler will produce an error if you try to initialize a type by passing less than the required values.

The `type` keyword can also be used for simply declaring aliases. These are useful for example for function types:

```typescript
type ValidatorFn =
  (c: FormControl) => { [key: string]: any }|null
```

this way we can avoid having to repeat the complex type definition every time we expect a value of that type:

```typescript
class FormControl {
  constructor (initialValue: string, validator: ValidatorFn | null) {
    ...
  }
}
```

TypeScript allows you to explicitly specify the properties at the definition of a class; additionally, it also provides access modifiers `public`, `private` and `protected`:

```typescript
class Person {
  public firstName: string;
  public lastName: string;
  private age: number;

  constructor (firstName: string, lastName: string, age: number) {
    ...
  }
}
```

Properties can also be defined in the constructor definition:

```typescript
class Person {
  constructor(
    public firstName: string,
    public lastName: string,
    private age: number
  ) {
    ...
  }
}
```

this means that the values passed as arguments of the constructor will automatically be assigned to the properties with the same names, much like C++'s initialization lists.

A class property can be declared as `readonly`, which works as `const`, which cannot be used with class properties though.

TypeScript supports interfaces, with the keyword `interface`:

```typescript
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}
```

Since TypeScript uses a *structural type system*, any object having properties named with those keys and of those types, can be assigned to a variable of type `Person`. For example:

```typescript
function savePerson (person: Person): void {
  ...
}

const p: Person = {
  firstName: "John",
  lastName: "Smith",
  age: 25
};
savePerson(p);
```

This would have worked even by not adding the `Person` type annotation to the object definition, since the object had the correct structure, matching the one defined by `Person`, anyway.

The `instanceof` operator is used to tell if the given value is an instance of a class. If we apply `instanceof` to a value that has the type of an interface, we would get a compile error, because that won't be an instance of a class. From another point of view, both `interface` and `type` don't produce any JavaScript code after compilation, but `instanceof` exists also in JavaScript, so we can only use `instanceof` with `class`, because that kind of code will fully be ported to JavaScript. `interface` and `type` should only be used to leverage the type checking of the TypeScript compilation. Additionally, since `interface` and `type` produce no output in JavaScript, we should declare as much as possible there, to have a smaller JavaScript object code. In general, finally, `types` have some more feature than `interface`.

In TypeScript two types are seen as the same if they have the same structure, meaning the same properties, with the same access and the same types, and methods with the same signatures. Additionally, a type is *assignable* to another if it has *at least* the same properties, meaning that it can have some more, and still can be used in place of the other

```typescript
class Person {
  name: String;
  age: number;
}

class Customer {
  name: String;
}
```

Here, `Person` can be used in place of `Customer`, because it satisfy the condition of having a property called `name` of type `String`. The same is not true the other way around, though.

We can define unions of structured types as well. In this case, however, we talk about *discriminated unions*, meaning that all types must have at least one property in common:

```typescript
class SearchAction {
  actionType = "SEARCH";
  ...
}

class SearchSuccessAction {
  actionType = "SEARCH_SUCCESS";
  ...
}

class SearchFailedAction {
  actionType = "SEARCH_FAILED";
  ...
}
```

Here, if we refer to a union type like `SearchAction | SearchSuccessAction | SearchFailedAction`, we are actually referring to a type that we know has the `actionType` property. This is important because in the code using this union I need to know I can access at least that property. Then, since the object is of a union type, we theoretically can expect it to have any of the properties of the types that take part to the union: however, we first must make sure which one the object actually is, before accessing specific properties. To check if a property of a union type is actually included in the given object, we can use the `in` keyword, like in:

```typescript
interface A { a: number };
interface B { b: string };

function foo (x: A | B) {
  if ("a" in x) {
    ...
  }
}
```

When a variable is declared as `unknown`, you'll be required to narrow its type down before proceeding:

```typescript
type Person = {
  address: string
}

let person: any;
person = JSON.parse('{ "adress": "25 Broadway" }');
console.log(person.address);
```

Here we misspelled "address" in the JSON string, so the program will print out `undefined`. The compiler wasn't able to spot the issue because `person` was declared of type `any` instead of `Person`. If we instead do:

```typescript
let person: unknown;
person = JSON.parse('{ "adress": "25 Broadway" }');
console.log(person.address)
```

here the compiler will produce an error, because we're trying to use an `unknown` variable. We could check if the attribute that we want is present:

```typescript
if ("address" in person) {
  console.log(person.address)
}
```

however this wouldn't satisfy the compiler still, because this check can only be done at runtime, meaning that the compiler still wouldn't know if `person` can indeed represent a `Person`. Turns out that TypeScript allows us to use this runtime check to satisfy the compiler at the same time, using a *type guard*:

```typescript
const isPerson = (object: any): object is Person => "address" in object;

if (isPerson(person)) {
  console.log(person.address)
}
```

here the return type of the function is a *type predicate*, which tells the compiler that the boolean being returned by that function will be enough to tell whether that object is of type `Person` or not; in other words, a type guard reassures the compiler that after running that function we will have resolved the type of the object (or at least if it's what we expect).<sup>1</sup>

## Object-Oriented Programming

TyepScript supports subclassing using the `extends` keyword:

```typescript
class Person {
  firstName = '';
  lastName = '';
  age = 0;
}

class Employee extends Person {
  department = '';
}

const empl = new Employee();
```

TypeScript supports the access modifiers `public`, `protected` and `private`, which work like in the other common OO languages.

TypeScript supports static class members:

```typescript
class Gangsta {
  static totalBullets = 100;

  shoot() {
    Gangsta.totalBullets--;
    console.log(`Bullets left: ${Gangsta.totalBullets}`);
  }
}
```

Of course every instance of `Gangsta` shares the same allocation of `totalBullets`, but instances of subclasses get their own allocation of static members.

The `super()` method invokes the constructor of the superclass:

```typescript
class Person {
  constructor (
    public firstName: string,
    public lastName: string,
    private age: number
  ) { }
}

class Employee extends Person {
  constructor (
    firstName: string,
    lastName: string,
    age: number,
    public department: string
  ) {
    super(firstName, lastName, age);
  }
}
```

On the other hand, the `super` keyword is used to access the invocation object of the parent class:

```typescript
class Person {
  constructor (
    public firstName: string,
    public lastName: string,
    private age: number
  ) { }
  sellStock(symbol: string, numberOfShares: number) {
    console.log(`Selling ${numberOfShares} of ${symbol}`);
  }
}

class Employee extends Person {
  constructor (
    firstName: string,
    lastName: string,
    age: number,
    public department: string
  ) {
    super(firstName, lastName, age);
  }

  sellStock(symbol: string, shares: number) {
    super.sellStock(symbol, shares);
  }
}
```

TypeScript supports abstract classes with the keyword `abstract`:

```typescript
abstract class Person {
  ...
  abstract increasePay(percent: number): void;
  ...
}
```

TypeScript supports method overloading, using this syntax:

```typescript
class ProductService {
  getProducts(description: string): Product[];
  getProducts(id: number): Product;
  getProducts(product: number | string): Product[] | Product {
    if (typeof product === 'number') {
      console.log(`Getting the product info for ${product}`);
      return { id: product, description: 'great product' };
    } else if (typeof product === 'string') {
      console.log(`Getting product with description ${product}`);
      return [
        { id: 123, description: product },
        { id: 789, description: product }
      ];
    } else {
      return {
        id: -1,
        description: 'Error: getProducts() accepts only number or string as args'
      };
    }
  }
}
```

For overloaded methods to work, we need to provide all overloaded methods with no implementation, and only one with the implementation that works for all cases. The arguments and return type of the last one must support all the arguments and return types of the previous. Of course we could just write the last implementation, discarding the overloaded definitions: these are helpful, though, for the TypeScript compiler to know what return type is correct for the given argument types, and thus it can provide better coherence check.

Method overloading is best employed with constructors, because constructors cannot be given different names, unlike methods:

```typescript
class Product {
  id: number;
  description: string;

  constructor();
  constructor(id: number);
  constructor(id: number, description: string);
  constructor(id?: number, description?: string) {
    ...
  }
}
```

## Other references

- [1](https://rangle.io/blog/how-to-use-typescript-type-guards/)
