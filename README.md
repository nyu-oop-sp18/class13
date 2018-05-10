# Class 13


## JavaScript

Javascript is a dynamic, weakly typed, prototype-based, multi-paradigm, high-level, interpreted programming language.
It is basically the language that the internet is built on. 
It has some features ("JavaScript is ridiculously liberal in what it allows" according to [Marijn Haverbeke](https://eloquentjavascript.net/00_intro.html)) that might be of interest after a semester of looking at Scala.


### Imperative

JavaScript is an imperative langauge that borrows most of its syntax from Java, and should be familiar to anyone who has coded in C/C++.


### Objects

JavaScript is based on *objects* - collection of name-value pairs.
Names are strings, and values can be any JavaScript value, including other objects.
Pretty much everything, apart from some primitive values such as numbers, strings, and booleans, is an object in JavaScript.

```javascript
// Creating an object
var person1 = {
  first: "Zaphod",
  last: "Beeblebrox",
  // Properties can be functions
  speak: function () { console.log("Woah!"); }
};

// Accessing properties of an object
console.log(person1.first);  // Zaphod
console.log(person1["last"]);  // Beeblebrox
person1.speak();  // Woah!

// Adding new properties to an object
person1.age = 999;
console.log(person1.age);  // 999

// Creating an empty object
var person2 = {};
console.log(person2.age);  // undefined
```


### Dynamic Typing

In JavaScript, *values* are typed, not *expressions*.

This means that at compile time, one cannot tell the type of a variable or a source expression.
How does JavaScript know whether a function can be applied on a value?
It checks the type of the value at run-time.

This allows us to write code like this:

```javascript
var x = 1;

console.log(x + 2);  // 3

x = 'hell';  // No errors here

console.log(x + 'o');  // hello
```

Why would we want to do that?
Static vs dynamic languages is one of the cat-vs-dogs style controversies that will probably never be settled.
You can spend hours reading the opinions of both sides on the internet.
Here are some common reasons people give for preferring dynamic languages:

- Quicker to hack together small programs: you can write more succinct code (for example see function `padLeft` below), and you don't have to wait for your code to compile/type-check.

- Less boilerplate: you do not need to write class definitions and type annotations.

- Better polymorphism: for example, [abstractions in Clojure](http://www.infoq.com/presentations/Clojure-The-Art-of-Abstraction).

On the other hand, here are some reasons for static typing:

- Better IDE support.

- Better performance: no need to check types at run-time; more opportunities for compiler optimizations.

- Some errors are caught early in the development process, potentially saving a lot of money.

- Refactoring and maintaining code is easier.

To simplify matters, dynamic languages seem to be better for quick, small projects, but do not scale as well to large, long-term projects.
This is one of the reasons why gradual or optional typing is becoming fashionable: people want to use languages like JavaScript for more serious work but realize that it doesn't scale well without adhering to a static typing discipline.
We will see below one attempt to do optional typing for JavaScript: TypeScript.


### Duck Typing

How does dynamic typing deal with objects? **Duck typing**.

What is duck typing?

> If it looks like a duck, swims like a duck, and quacks like a duck, then it probably *is* a duck.

In other words, JavaScript allows you to call a function `f` on any object `o` as long as all the properties of `o` that `f` needs are present in `o`.

```javascript
var duck = {
  quack: function () { console.log("Quack quack!"); },
  canSwim: true,
}

var hen = {
  cluck: function () { console.log("Cluck cluck!"); },
  canSwim: false,
}

function dunk(a) {
  if (!a.canSwim)
    console.log("Whoops, this animal drowned!");
  else
    a.quack();
}

dunk(duck);  // Quack quack!
dunk(hen);  // Whoops, this animal drowned
```

In the code above, the call `dunk(hen)` is allowed even though `hen` does not have a `quack` property, because that line was never reached in run-time.


### Classes?

JavaScript does not organize objects using classes.
If you want to have a uniform way of creating a whole bunch of objects to represent people, you would use a function to create person objects (a kind of a person factory):

```javascript
function makePerson(first, last) {
  return {
    first: first,
    last: last,
    fullName: function() {
      return this.first + ' ' + this.last;
    },
  };
}

p1 = makePerson("Dodo", "Baggins");
p2 = makePerson("Brodo", "Baggins");
console.log(p1.fullName());  // Dodo Baggins
```

The `this` keyword, used inside a funciton, refers to the current object.
Since in the final line `fullName` is called as `p1.fullName()`, `p1` is the current object.

JavaScript gives you a simpler, and more familiar, way of creating object factories:

```javascript
function Person(first, last) {
  this.first = first;
  this.last = last;
  this.fullName = function() {
    return this.first + ' ' + this.last;
  };
}
var p1 = new Person('Hairy', 'Porter');
console.log(p1.fullName());  // Hairy Porter
```

Essentially, `new` first creates an empty object, and then calls the `Person` function with `this` pointing to the new object.
Notice that the `Person` function does not return anything -- it is `new` that returns a reference to the new object and stores it in `p1`.


### Prototypes

JavaScript uses *prototypes* as a way to model inheritance and other relations between classes.
Every object has a prototype, which is a reference to another object (or `null`).
For example, if you create a new object
```javascript
var o = { foo: "123" };
```
then JavaScript automatically sets the prototype of `o` to `Object.prototype`.
What is the prototype of `Object.prototype`, you may ask? `null`.

Every time we access a property of an object, `o.p`, JavaScript will first look for `p` in `o`, and failing that, will look in the prototype of `o`, and then the prototype of the prototype of `o`, and so on.
This is called the *prototype chain*, and the search for `p` ends when the prototype of some object is `null`.


To understand the use of prototypes, consider the `Person` example above.
Currently, every `Person` object has its own copy of the function `fullName`, wouldn't it be better if they could all share a single one?
This can be done by defining `fullName` in the prototype of `Person`:

```javascript
function Person(first, last) {
  this.first = first;
  this.last = last;
}

Person.prototype.fullName = function() {
  return this.first + ' ' + this.last;
};

var p1 = new Person('Dom', 'Bill-Door');
var p2 = new Person('Grand', 'Elf');
console.log(p1.fullName());  // Dom Bill-Door
console.log(p2.fullName());  // Grand Elf
```

`Person.prototype` is an object that is shared by all instances of `Person`.
When you use the syntax `var o = new Person(...)` to create an object, JavaScript automatically sets the prototype of `o` to `Person.prototype`.

In the example above, the prototype chains for `p1` and `p2` look like this:
```
p1 ---> Person.prototype ---> Object.prototype ---> null
p2 ---> Person.prototype ---> Object.prototype ---> null
```
Thus, the calls to `p1.fullName` and `p2.fullName` both get resolved by following the prototype chain up one level to `Person.prototype` and using the `fullName` function defined there.

Protypes are more powerful than classes when it comes to modeling the relations between objects.
One can, for instance, model classes quite easily using prototypes, as we see below.


### Inheritance

The simplest way to implement inheritance in JavaScript is to link the prototype of an object of the subtype to the object of a supertype.

```javascript
var a = {
  color: "blue",
  size: 12,
  toString: function() { return "a"; }
}

var b = Object.create(a);  // Creates an object and sets b's prototype to a

// Override some properties
b.color = "red";
b.toString = function () { return "b"; };

// These properties use b's properties
console.log(b.color);  // red
console.log(b.toString());  // b

// This property is inherited from a
console.log(b.size);  // 12
```

The prototype chain for `b` now looks like:
```
b ---> a ---> Object.prototype ---> null
```

You can also do the same thing with the factory/constructor function syntax:

```javascript
function A() {
  this.color = "blue";
  this.size = 12;
}

A.prototype.toString = function() { return "a"; };

function B() {
  A.call(this);
  this.color = "red";
}

B.prototype = Object.create(A.prototype);
// Creates a new object whose prototype is A.prototype

B.prototype.toString = function() { return "b"; };

var b = new B();

// These properties use b's properties
console.log(b.color);  // red
console.log(b.toString());  // b

// This property is inherited from a
console.log(b.size);  // 12
```

In this case, the prototype chain for `b` looks like:
```
b ---> B.prototype ---> A.prototype ---> Object.prototype ---> null
```

Newer versions of JavaScript give you a more familiar `class` syntax to do the same thing:

```javascript
class A {
  constructor() {
    this.color = "blue";
    this.size = 12;
  }
  
  toString () { return "a"; }
}

class B extends A {
  constructor () {
    super();
    this.color = "red";
  }
  
  toString () { return "b"; }
}

var b = new B();

// These properties use b's properties
console.log(b.color);  // red
console.log(b.toString());  // b

// This property is inherited from a
console.log(b.size);  // 12
```

Note that there is the inheritance hierarchy is not static in JavaScript.
In fact, in some implementations of JavaScript, you can modify the prototype chain of an object on-the-fly!

```javascript
class A {
  foo () { return "a"; }
}

class B extends A {}

var b = new B();

// Prototype chain:
// b ---> B.prototype ---> A.prototype ---> Object.prototype ---> null

console.log(b.foo());  // a

class C {
  foo () { return "c"; }
}

b.__proto__.__proto__ = C.prototype;  // Deprecated and not always supported - don't use!

// Prototype chain:
// b ---> B.prototype ---> C.prototype ---> Object.prototype ---> null

console.log(b.foo());  // c
```

Another important thing to note is that you do not get the security properties provided to you by classes in an OOP language.
Duck typing extends to methods (which are just functions) and which can be swapped out by different implementations on-the-fly during execution.
In fact, you can even do this with prototype objects (which is a fun way of injecting malicious code into libraries).
For example, suppose you have a common library that defines a class `A`:

```javascript
class A {
  foo () { console.log("Do something benign and hopefully useful."); }
}
```

And there is some application code which you want to hack:

```javascript
...
var a = new A();
a.foo();  // Do something benign and hopefully useful.
...
```

And you (the evil hacker) manage to execute some code before the harmless application code runs:

```javascript
C.prototype.foo = function () { return "Do something harmful and malicious."; };
```

Then the application is now bent to your evil purpose:

```javascript
...
var a = new A();
a.foo();  // Do something harmful and malicious.
...
```

### More about Methods

Methods are nothing more than properties of objects that hold function values (recall, functions are objects too).
JavaScript, being dynamically typed, has no concept of method signatures.
This means there is no method overloading; in fact, since object properties are identified by their names, there can only be one method of a given name in an object.

However, a method in one object can be overridden by a definition of a method of the same name in an object further down the prototype chain, as done for the `toString` function above.
As we will see below, a consequence of the prototype chain and dynamic duck/structural typing is that there is no extra support needed for dynamic dispatch.



Remember this Scala example?

```scala
class A(val x: Int, val y: Int) { 
  def m1() = { ... }
  def m2() = { ... }
}

class B(x1: Int, y1: Int, val z: Int) extends A(x1, y1) {
  override def m2() = { /* overriding A.m2 */ ... }
  def m3() = { ... }
}
```

Scala implements these using vtables and vpointers as follows:

```
      A Instance:                A vtable:
    0┌─────────────┐        ┌> 0┌────────────┐                    ┌─────────────┐
     │ vptr        │────────┘   │ ptr. to m1 │───────────────────>│impl. of A.m1│
    8├─────────────┤           8├────────────┤                    └─────────────┘
     │ value of x  │            │ ptr. to m2 │────────┐  ┌─────────────┐   ^
   12├─────────────┤            └────────────┘        └─>│impl. of A.m2│   │
     │ value of y  │                                     └─────────────┘   │
     └─────────────┘                                                       │
                                                                           │
      B Instance:                B vtable:                                 │
    0┌─────────────┐        ┌> 0┌────────────┐                             │
     │ vptr        │────────┘   │ ptr. to m1 │─────────────────────────────┘ 
    8├─────────────┤           8├────────────┤           ┌─────────────┐
     │ value of x  │            │ ptr. to m2 │──────────>│impl. of B.m2│
   12├─────────────┤          16├════════════┤           └─────────────┘ 
     │ value of y  │            │ ptr. to m3 │────────┐  ┌─────────────┐
   16├═════════════┤            └────────────┘        └─>│impl. of B.m3│
     │ value of z  │                                     └─────────────┘
     └─────────────┘
```

However, we cannot implement method lookup using optimizations such as using offsets or vtables in JavaScript.
Why not?
Because in JavaScript, properties of objects can be referenced using string values, including statically-unknown values:

```javascript
var o = {
  foo: 1,
  bar: 2,
}

if (Math.random() < 0.5)
  str = "foo";
else
  str = "bar";

console.log(o[str]);  // Is this accessing o.foo or o.bar?!
``` 

Thus there is no way to calculate offsets for different properties.
Of course, another reason is the fact that there are no classes, so for every object in your program you'd potentially need a separate offset table!

However, in JavaScript, you don't really need to do all this, because the prototype model allows you to write programs that have similar lookup costs.
In JavaScript, our example looks like (not using the class syntax for clarity):

```javascript
function A(x, y) {
  this.x = x;
  this.y = y;
}

A.prototype.m1 = function() { };
A.prototype.m2 = function() { };

function B(x, y, z) {
  A.call(this, x, y);
  this.z = z;
}

B.prototype.m2 = function() { };  // overriding A.m2
B.prototype.m3 = function() { };
```

And the underlying implementation looks like (some JavaScript engines use more [complex implementations](https://github.com/v8/v8/wiki/Design%20Elements) for performance reasons):

```
 A instance:              A.prototype:         +───> Object.prototype
+───────────────+        +───────────────+     |
| [[Prototype]] +──+───> | [[Prototype]] +─────+
+───────────────+  |     +───────────────+           +───────────────+
| x             |  |     | m1            +─────────> | impl. of A.m1 |
+───────────────+  |     +───────────────+           +───────────────+
| y             |  |     | m2            +─────+
+───────────────+  |     +───────────────+     |     +───────────────+
                   |                           +───> | impl. of A.m2 |
                   +─────────────────────────+       +───────────────+
                                             |
 B instance:              B.prototype        |
+───────────────+        +───────────────+   |
| [[Prototype]] +──────> | [[Prototype]] +───+
+───────────────+        +───────────────+           +───────────────+
| x             |        | m2            +─────────> | impl. of B.m2 |
+───────────────+        +───────────────+           +───────────────+
| y             |        | m3            +─────+
+───────────────+        +───────────────+     |     +───────────────+
| z             |                              +───> | impl. of B.m3 |
+───────────────+                                    +───────────────+
```


### Functional Programming

Since functions are also first-class objects, you can write (with a little bit of hoop-jumping) functional programs in JavaScript.

```javascript
function map(f, l) {
  for (var i = 0; i < l.length; i++)
    l[i] = f([l[i]]);
}

function sq(x) { return x*x; }

l = [1, 2, 4, 8];

console.log(l);  // [ 1, 2, 4, 8 ]
map(sq, l);
console.log(l);  // [ 1, 4, 16, 64 ]
```

You can also force values and objects to be immutable as follows.

```javascript
const o4 = Object.freeze({ foo: 'never going to change me' });

// Cannot mutate the object
// o4.foo = 'talk to the hand' // Error!

// Cannot reassign the variable
// o4 = { message: "ain't gonna happen, sorry" }; // Error
```

However, if you care about performance, then consider using `Immutable.js`.
`Immutable.js` implements Lists, Stacks, Maps, Sets, and other data structures using persistent data structures. 

For more streamlined support for functional programming, take a look at the [Ramda](http://ramdajs.com/) and [Lodash](https://lodash.com/) libraries.

## TypeScript

TypeScript is an attempt to tame the JavaScript jungle using static typing.
It is essentially a *strict superset* of JavaScript: any valid JavaScript code is valid TypeScript code.
TypeScript adds syntax that allows the developer to annotate the code with the types of various expressions.

To make it easier for developers to port existing JavaScript codebases, (a) TypeScript allows you to specify the types of certain variables to be `any`, and (b) even if TypeScript finds some static type errors, it still emits JavaScript.
The first property will be more clear when we look at the type system of TypeScript.

### TypeScript's (optional) types

TypeScript has the same basic types as JavaScript: `number`, `boolean`, and `string`.
TypeScript can automatically infer the types of certain expressions and warns you when you assign a variable a value of a different type.

```typescript
var x = 5;
x = "str";  // Error: Type '"asdf"' is not assignable to type 'number'.
```

You can also manually annotate a variable with a type, in which case TypeScript will prevent you from using it in contexts where another type is expected.
```typescript
var y: string;
var z = y * 1.5;  // Error: The left-hand side of an arithmetic operation must be of type 'any', 'number' or an enum type.
```

You can opt-out of type checking by marking an expression with the `any` type.
This is useful for the intermediate states in migrating a JavaScript program, or when using a 3rd party library that is not yet typed.
```typescript
var y: any;
var z = y * 1.5;  // OK
```

There are also types `null` and `undefined` whose only inhabitants are `null` and `undefined` respectively.
However, these are subtypes of all other types.
```typescript
var x: number;
var y: undefined;
x = y;  // OK
```

### Interfaces

TypeScript can also reason about the duck-typing or structural subtyping done by JavaScript.
To do this, TypeScript gives you *interfaces*, which are types that describe objects.
For example:

```typescript
function printName(o: { name: string }) {
  console.log(o.name);
}

var x = {
    name: "X",
    size: 12,
}

var y = {
    age: 43,
    height: 150,
}

printName(x);  // OK
printName(y);
// Error: Argument of type '{ age: number; height: number; }' is not assignable to parameter of type '{ name: string; }'.
// Property 'name' is missing in type '{ age: number; height: number; }'.
```

If you find yourself writing the same structural type over and over, you can declare it as an interface:

```typescript
interface NamedVal {
  name: string;
}

function printName(o: NamedVal) {
  console.log(o.name);
}

var y = {
    age: 43,
    height: 150,
}

printName(y);
// Error: Argument of type '{ age: number; height: number; }' is not assignable to parameter of type 'NamedVal'.
// Property 'name' is missing in type '{ age: number; height: number; }'.
```

One of the most common uses of interfaces in languages like C# and Java, that of explicitly enforcing that a class meets a particular contract, is also possible in TypeScript.

```typescript
class Person implements NamedVal {
  age: number;
  name: string;
  constructor(a, n) {
    this.age = a;
    this.name = n;
  }
}
```

### Union types

Suppose you have a function that expects either a number or a string:

```typescript
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // returns "    Hello world"
```

The problem with using the type `any` for `padding` is that TypeScript will not warn you when you execute something like `padLeft("sss", false)`.
In standard OO languages, we can add an appropriate class in the type hierarchy and use that as the type for `padding`, however this is both an overkill (you cannot do this for every such function), and is also bad for performance as you would have to "box" the primitive types `string` and `number` inside classes.
We can instead use a *union type*:

```typescript
function padLeft(value: string, padding: string | number) {
    // ...
}

let indentedString = padLeft("Hello world", true);  // Error
```

### Generics

How would you type the generic identity function?
Here is a first attempt:

```typescript
function identity(arg: any): any {
  return arg;
}

var x = 5;
var y = identity(x) + 1.3;  // y has type any
```

The problem here is that we have lost some type information.
We do not know that since `x` has type `number`, so does `identity(x)`.

TypeScript allows you to use *type variables* to describe the types of generic functions:

```typescript
function identity<T>(arg: T): T {
  return arg;
}

var x = 5;
var y = identity(x) + 1.3;  // y has type number
```

Of course, inside a generic function, you cannot assume anything about the generic type `T`:

```typescript
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

If we do want to impose such a constraint on `T`, we can use an interface:

```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so OK
    return arg;
}

loggingIdentity(3);  // Error, number doesn't have a .length property
```


### Unsoundness

A type system is said to be *sound* if every program that has no type errors will not have any type-related errors (of certain kinds) during run-time.
Unfortunately, TypeScript is unsound.
There are programs that TypeScript allows without any type errors during compile-time, which throw type errors in run-time.
This is intentional, for the TypeScript designers [chose](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Design-Goals) to prioritize productivity and a simpler type system over soundness.
However, it is good to know the situations in which TypeScript is unsound.
For example:

```typescript
class Animal {
   name: string;
   constructor(name: string) {
       this.name = name;
   }
}

class Dog extends Animal {
    bark() { console.log("woof"); }
}

class Cat extends Animal {
    purr() { console.log("prrr"); }
}

let cats: Array<Cat> = []; // can only contain cats
let animals: Array<Animal> = []; // can only contain animals

// wow, works, but is no longer safe
animals = cats;

animals.push(new Dog('Brutus'));

cats.forEach(cat => cat.purr());
// Uncaught TypeError: cat.purr is not a function
```


## Further Reading

JavaScript:

- [Eloquent JavaScript](https://eloquentjavascript.net/): a free online book

- [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS): free online book series

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain

- [Try JavaScript in your browser](https://repl.it/repls/SweetExhaustedBinarytree)

TypeScript:

- [The TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html)

- [TypeScript Deep Dive](https://basarat.gitbooks.io/typescript/content/)

- [Try TypeScript in your browser](https://www.typescriptlang.org/play/)
