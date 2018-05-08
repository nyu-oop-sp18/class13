# Class 13

## JavaScript

Dynamic language: JS

### Imperative

Like C/Java


### Objects

JS is based on *objects* - collection of name-value pairs.
Names are strings, and values can be any JS value, including other objects.
Pretty much everything, apart from some primitive values such as numbers, strings, and booleans, is an object in JS.

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
console.log(person1.age);  // undefined
```


### Dynamic Typing

In JavaScript, *values* are typed, not *expressions*.

This means that at compile time, one cannot tell the type of a variable or a source expression.
How does JS know whether a function can be applied on a value?
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
Here are some common reasons people give for preferring dynamic languages (warning: these do not necessarily reflect the views of the instructor, nor are they a recommendation):

- Quicker to hack together small programs.

- Less boilerplate: you do not need to write class definitions and type annotations.

- Better polymorphism: for example, [abstractions in Clojure](http://www.infoq.com/presentations/Clojure-The-Art-of-Abstraction).

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
    duck.quack();
}

dunk(duck);  // Quack quack!
dunk(hen);  // Whoops, this animal drowned
```

In the code above, the call `dunk(hen)` is allowed even though `hen` does not have a `quack` property, because that line was never reached in run-time.


### Classes?

JS does not organize objects using classes.
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

JS gives you a simpler, and more familiar, way of creating object factories:

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

JS uses *prototypes* as a way to model inheritance and other relations between classes.
Every object has a prototype, which is a reference to another object (or `null`).
For example, if you create a new object
```javascript
var o = { foo: "123" };
```
then JS automatically sets the prototype of `o` to `Object.prototype`.
What is the prototype of `Object.prototype`, you may ask? `null`.

Every time we access a property of an object, `o.p`, JS will first look for `p` in `o`, and failing that, will look in the prototype of `o`, and then the prototype of the prototype of `o`, and so on.
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
When you use the syntax `var o = new Person(...)` to create an object, JS automatically sets the prototype of `o` to `Person.prototype`.

In the example above, the prototype chains for `p1` and `p2` look like this:
```
p1 ---> Person.prototype ---> Object.prototype ---> null
p2 ---> Person.prototype ---> Object.prototype ---> null
```
Thus, the calls to `p1.fullName` and `p2.fullName` both get resolved by following the prototype chain up one level to `Person.prototype` and using the `fullName` function defined there.

Protypes are more powerful than classes when it comes to modeling the relations between objects.
One can, for instance, model classes quite easily using prototypes, as we see below.


### Inheritance

The simplest way to implement inheritance in JS is to link the prototype of an object of the subtype to the object of a supertype.


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

Newer versions of JS give you a more familiar `class` syntax to do the same thing:

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

### Dynamic Dispatch

JavaScript, being dynamically typed, has no concept of method signatures.
This means there is no method overloading; in fact, there can only be one function of a given name in an object.
However, a method in one object can be overridden by a definition of a method of the same name in an object further down the prototype chain, as done for the `toString` function above.
This also means that there is no extra support needed for dynamic dispatch.
In JS, this is simply a consequence of the prototype chain and dynamic duck/structural typing.


Remember this Scala example?

```scala
class A(val x: Int) {
  def m(): Int = x
}
class B(x0: Int, val y: Int) extends A(x0) {
  override def m(): Int = x + y
}

def f(a: A) = a.m()

val a: A = new B(1, 2)
a.m()  // 3
```

In JavaScript, this looks like:
```javascript
class A {
  constructor(x) {
    this.x = x;
  }
  
  m () { return this.x; }
}

class B extends A {
  constructor (x, y) {
    super(x);
    this.y = y;
  }
  
  m () { return this.x + this.y; }
}

var b = new B(1, 2);
console.log(b.m());  // 3
```

### Functional Programming

Since functions are also first-class objects, you can write (with a little bit of hoop-jumping) functional programs in JavaScript.

```javascript
function map(f, l) {
  for (var i = 0; i < l.length; i++)
    l[i] = f.apply(null, [l[i]]);
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

TypeScript is an attempt to tame the JS jungle using static typing.
It is essentially a *strict superset* of JS: any valid JS code is valid TS code.
TS adds syntax that allows the developer to annotate the code with the types of various expressions.

To make it easier for developers to port existing JS codebases, (a) TS allows you to specify the types of certain variables to be `any`, and (b) even if TS finds some static type errors, it still emits JS.
The first property will be more clear when we look at the type system of TS.

### TypeScript's (optional) types

TS has the same basic types as JS: `number`, `boolean`, and `string`.
TS can automatically infer the types of certain expressions and warns you when you assign a variable a value of a different type.

```typescript
var x = 5;
x = "str";  // Error: Type '"asdf"' is not assignable to type 'number'.
```

You can also manually annotate a variable with a type, in which case TS will prevent you from using it in contexts where another type is expected.
```typescript
var y: string;
var z = y * 1.5;  // Error: The left-hand side of an arithmetic operation must be of type 'any', 'number' or an enum type.
```

You can opt-out of type checking by marking an expression with the `any` type.
This is useful for the intermediate states in migrating a JS program, or when using a 3rd party library that is not yet typed.
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

TS can also reason about the duck-typing or structural subtyping done by JS.
To do this, TS gives you *interfaces*, which are types that describe objects.
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

### Union and Intersection types

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

The problem with using the type `any` for `padding` is that TS will not warn you when you execute something like `padLeft("sss", false)`.
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

TS allows you to use *type variables* to describe the types of generic functions:

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

## Further Reading

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain

- [Try JavaScript in your browser](https://repl.it/repls/SweetExhaustedBinarytree)

- [The TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html)

- [Try TypeScript in your browser](https://www.typescriptlang.org/play/)
