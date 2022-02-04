# Typescript Fundamentals

## Bootstrapping the simplest JS project only with Typescript

1. Initialise the project with the command `npm init` (or create the package.json manually)
2. Add typescript as a development dependency (`npm install typescript -D`)
3. Create npm script to compile typescript into vanilla JS. It will speed up development (and convenience). 
In the package.json `scripts` property declare the new script name as `dev` with the value `tsc` (`"dev": "tsc --watch --preserveWatchOutput"`). To execute run `npm run dev`.
To run inline without the npm script, simply use the `npx` command with the same parameters (`tsc --watch --preserveWatchOutput`)
4. Create the typescript configuration file (tsconfig.json) with the command `tsc --init` (`npx tsc --init`).
The most important attributes, at least at first, are `compilerOptions.outDir`, `compilerOptions.target` and `include`.
    1. If the project is meant to run with node, add the `compilerOptions.module` with the value `commonjs`. To run the generated source simply execute `node {generated_file}.js`.
5. Enable tsc to generate type definition if the project is meant to be a library so that other modules can make use of the types. Properties `declaration` and `declarationMap` in the tsconfig.json


## Concepts

### TSC compiler

When TSC compiles typescript files into javascript files, in a nutshell* it (1) strips away the types so that the generated source is runnable by js engines and is compatible with projects and modules that are not using typescript and (2) generates a new declaration file (.d.ts) that can be used by projects that are using typescript to make use of the defined constraints - these projects sort of "reassemble" the genarated JS file and declaration.

*it also converts features that are not supported by JS into something that is supported - depends on the target and module configs.

`.js` files contain only code that run
`.ts` files contain both type information and code that runs
`.d.ts` files contain only type information


### Variables and Values

Typescript is capable of inferring the variable types when their values are assigned at initialization time:

```
let a = 1; // let a: number
a = 'abc'; // error `type string is not assignable to type number`
const b = 6; // const b: 6 - this is known as literal types - it can only be 6 and can never be changed
```

When a variable is not initialized with a value typescript assigns the type `any` but it greatly weakens the source base as any could break up all the type definition:

```
let c; // let c: any
c = 0;
c = new Date();
new Promise(c); // promise does not take a date not a number as parameter, but typescript is perfectly happy with it as it is not capable of evaluating the variable type (`any`). 

to fix it it is possible to assign the type via the colon syntax:

let c: Date;
c = 0; // error `type number is not assignable to type date`.
new Promise(c); // error, promise does not take a date
```

Typescript is also capable of inferring the return type in many cases:

```
function sum(a, b) {
  return a + b; // in this case this function is adding two unknown types (or anys) that happens to be numbers
}

new Promise(sum(1, 2)); // no error, no guarantee

function sum(a: number, b: number) {
  return a + b; // in this case typescript is capable of inferring the type because the result of adding two numbers will always be a number;
}

new Promise(sum(1, 2)); // compilation error
```

It might be a good idea to only explicitly declare the type when typescript cannot infer it (does not apply for function's return type nor arguments - would it be better to surfice the problem where the function is declared or where it is used?). It helps to keep the source base cleaner and also helps when refactoring with less things to change.


### Objects

Objects consist of properties and the type those properties can be assigned is similar to regular variables. 

```
let car: {
  make: string
  model: string
  year: number
  chargeVoltage?: number // optional
}

car.year = 'abc' // error
```

Functions arguments can have the structure of the object which it receives defined in line as though it was a primitive:

```
function evaluateCar(car: {
  make: string
  model: string
  year: number
  chargeVoltage?: number
}): number {
  return 0;
}

evaluateCar({make: 'Toyota', model: 'Corolla', year: 2020});
```

#### Type Guard

Typescript is smart enough to eliminate the possibility of an optional property being undefinied when a undefinied check is performed before accessing the property.

```
  car.chargeVoltage.toFixed(1234) // compilation error
  
  if (car.chargeVoltage)
    `${car.chargeVoltage.toFixed(1234)}` // no compilation error as the chargeVoltage presence is being checked beforehand.
```

#### Excess property checking

Typescript ensures that object literals only contain the necessary properties and nothing else.
If any extra property is passed with an object literal type script will give a compilation error.

```
evaluateCar({make: 'Toyota', model: 'Corolla', year: 2020, colour: 'RED'});
```

The reason for that is that as the object literal is passed directly to the function (and the function does not declare that property) and is not reusable anywhere else, that property will never be accessible neither by the function itself nor by whatever subfunction this object may be passed - nobody has access to it.

To solve this issue one of the following solutions could be applied:

1. Remove the colour property from the object
2. add the colour property to the function argument type
3. Instead of declaring the object inline as an object literal, create the object beforehand and pass to the function as parameter (in this case the colour is not accessible by evaluteCar function but it may be accessible in other places) 

```
  const car = {make: 'Toyota', model: 'Corolla', year: 2020, colour: 'RED'};
  evaluateCar(car); // not accessing inside this function scope
  car.colour; // but accessible outside - and even other functions might declare colour as part of their function argument type
```

#### Index Signatures 

Typescript allows storage of a consistent type of value under an arbitrary key (when the object property is not rigid) via what is called index signatures. Example:

```
const phones = {
  home: {country: "+44", number: "827-123-111"},
  work: {country: "+44", number: "827-123-111"}
}
```

Type definition:

```
const phones: {
  [k: string]: {
    country: string
    number: string
  }
} = {} // just initialised with an empty object
```

#### Arrays

Typescript is capable of inferring the type of elements of an array, even when it is a complex object.

```
const arr1 = ["a", "b"] // const arr1: string[]

const cars = [{make: 'Toyota', model: 'Corolla'}] // const cars: [{make string; model string;}]
```

However in case one wants to be explicit, it is possible to define the type upfront:

```
const arr1: string[] = ["a", "b"]

const cars: {
  make: string
  model: string
}[] = [{make: 'Toyota', model: 'Corolla'}]
```

#### Tuples

Typescript tries to make assumptions when determining types so that it provides type safety but not getting in the way of the developer.
For tuples typescript assumes that the type is a heterogeneous array rather than a tuple to prevent imposing restrictions in unwanted situations - it is not possible to determine if the intended type was an array or a tuple due to the similarities in the declaration.


```
const car = ["Toyota", "Corolla", 1997]; // const car: (string | number)[]
const [make, model, year] = car // const year: string | number - as it is an array of number or string any of this could be in those positions, thus it is not possible to infer the type when destructuring 
```

The way to declare tuples properly is to do so explicitly - so that position and types are inforced

```
const car: [string, string, number] = ["Toyota", "Corolla", 1997]; // const car: [string, string, number]
const [make, model, year] = car // now typescript is capable of confidently telling what type is in each position
```

Caveats: even though typescript enforces tuple size at assignment, it does not inforces anything afterwards (just types) - it is possible to pop and push values at wish.
It happens because there is no way equivalence checks when invoking the operation methods.

```
car.push('asd')
car.push('asd')
```

#### Union Types - Narrowing with type guards

Union types are defined using a pipe:

```
const colour: 'red' | 'black' = 'red';
```

When using union with tuples, for instance to return the result and content of a processing, initially typescript only allows access to what is safe and guaranteed to be shared between the objects and structures.
In order to be able to access specific properties, it is necessary to narrow down the result via type guard - unless typescript is capable of defining the content when assigning by inference.

```

function fetchUserData(): ['error', Error] | ['success', {name: string; email: string;}] {
  return ['success', {name: 'Mary', email: 'test@gmail.com'}]
}

const outcome = fetchUserData();
outcome[1].name; // only name is accessibe as it is a common property between the inline object and error (and even their types match)

// this concept is known as discriminated unions or "tagged" union type
// typescript is capable of performing this kind of type guarding thank to the predetermined keys that could be stored in the first position of the tuple ('error' | 'success). 
if (outcome[0] === 'error') {
  outcome[1] // here typescript treats this object as an error due to the control flow via type guard 
} else {
  outcome[1].email // here typescript assumes that it could be anything that didn't enter the first condition, thus it could only be that object structure
}

// another way of narrowing down the types is with instanceof, however considering that typescript is rather has a structural rather than nominal type system, it could be quite limiting in some situations 

const [result, content] = outcome
if (content instanceof Error) {
  outcome[1] // here typescript treats this object as an error due to the control flow via type guard 
} else {
  outcome[1].email // here typescript assumes that it could be anything that didn't enter the first condition, thus it could only be that object structure
}

```

Intersection types are defined using a ampersand. It behaves as a pseudo inheritance for types.

```

function test(): Date & { end: string} { // returns an object which contains everything from Date and a string property named end
  return {...new Date, end: 'abc'}
}

type car = { // types one of the very few cases where the type definition (information) is defined on the right side of the assignment operator 
  make: string
  model: string
}

type truck = car & {
  size: number
}

```

#### Type aliases 

Type aliases are meant to give convenient names for types. Instead of keep repeating the same type all over the place (on variable, arguments, function results, etc), it is better to have a centralised definition and simply reference it.

```
// @types.ts
export type car = {
  make: string
  model: string
}

// @index.ts
import { Car } from './types';

const newCar: Car = {
  make: 'Toyota',
  model: 'Corolla'
}

```

By convention type aliases must be named using the Title pattern.

There can never two type aliases with the same names in the same scope - this is a typescript limitation and it does not apply to interfaces.

#### Interfaces

Interfaces are more limited than type aliases and they can only be used to define object types. Union type operator does not work with interfaces as the result would be something that is not an object type (I have these two possibilities and the result is one or another - interfaces cannot describe something like that).

Inheritance - typescript calls keywords like extends and imports `heritage clauses`. These keywords are used to describe ancestry in an object oriented hierarchy.

`Extends` is used to describe inheritance between like things - classes can extend from classes and interfaces from interfaces.
`Implements` is used to describe inheritance between unlike things - classes can implement interfaces.

```
interface Animal {
  name: string
  eat(food: string): void
}

interface Canine extends Animal {
  bark(): void;
}

class Dog implements Canine {
  name: string
  eat(food: string) {}
  bark() {}
}

class Pug extends Dog {

}
```

Even though it is possible for classes to extend from type aliases it is generally not a good a idea. Types more ofter than not can become a union type, what would break the source base as the object oriented inheritance does not support it.

```
// this case works
type Animal = {
  name: string
  eat(food: string): void
}

class Dog implements Animal {
  name: string
  eat(food: string) {}
  bark() {}
}

// but this does not
type Animal = {
  name: string
  eat(food: string): void
} | string

class Dog implements Animal {
  name: string
  eat(food: string) {}
  bark() {}
}
```

###### Interfaces are OPEN in javascript

It means that every time that an interface is defined with the same name of an existing interface, all that happens is that the new definitions are added on top of the previous one(s). No matter the place. It is not localized.

Example:

```
interface Dog {
  bark(): void;
}

function test2(dog: Dog) {
  dog.bark(); 
  dog.eat(); // it is available and functional
}

interface Dog {
  eat(): void;
}
```

It is possible to augment even interfaces from the javascript api (for instance the Window interface).

##### Interfaces or Type aliases

Generally there is no advantage or disadvantage using one or another. There are few situations that one might make more sense than the other:
1. When you need to define something other than an object (e.g when you have to use union to declare that a function returns a number or (|) an error), it makes more sense to stick with types
2. If it is something meant to be consumed by a class (Dog extends Animal), it makes more sense to stick with interfaces
3. If clients and consumers of your types must be able to augment them, then you must use interfaces. 

##### Recursive Types

When you need to define an object that self-reference itself all you have to do is the following:
```
type NestedNumbers = number | NestedNumbers[];

const val: NestedNumbers = [3, 4, [4, 3, [8], 3], 3, [1, 2]];

// type guard to eliminate the possibility of val being just a number
if (typeof val !== 'number') {
  val.push(123)
  val.push([41, 123]);
  val.push("this does not work");
}

```


##### Notes

Type systems:
static vs dynamic type systems
nominal vs structural types

Duck typing = if it looks like a duck, swims like a duck and quacks like a duck it is probably a duck - another way of describing dynamic typing.

Strong type and Weak type are terms generally used to refer to static and dynamic types but there is no concensus to what it really means.