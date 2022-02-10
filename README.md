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
// index signature
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

#### Callable types

Both interfaces and type aliases offer the capability to descibre call signatures:

```
interface TwoNumbersCalc {
  (x: number, y: number): number
}

const add: TwoNumbersCalc = (a, b) => a + b // even though it is not possible to infer argument types (and sometimes return), in this case typescript is capable of defining the types thanks to the interface

type TwoNumbersCalc = (x: number, y: number) => number
const subtract: TwoNumbersCalc = (a, b) => a - b

```

##### void x undefined

There are some succinct differences between void and undefinied. Functions that declare their return as `void` are telling that their return should be ignored - and actually that return could be anything. Functions that declare their return as `undefined` are telling that they actually return undefined, not anything else.

#### Function overloading
Sometimes it might be necessary to create functions that handle very high level operations and receive multiple arguments that have some sort of dependency between them. Using union type would not impose any kind of restriction to those dependencies.

Imagine a function that handles events of iframes and forms (FormSubmitHandler and MessageHandler) and also receives the related elements (HTMLFormElement and HTMLIFrameElement respectively). The combinations `FormSubmitHandler and HTMLIFrameElement` and `MessageHandler and HTMLFormElement` wouldn't make sense. To help typescript understand that dependency and enforce it, it is necessary to use the function overloading feature:

```
type FormSubmitHandler = (data: FormData) => void
type MessageHandler = (evt: MessageEvent) => void

function handleMainEvent(
  elem: HTMLFormElement,
  handler: FormSubmitHandler
)
function handleMainEvent(
  elem: HTMLIFrameElement,
  handler: MessageHandler
)
function handleMainEvent(
  elem: HTMLFormElement | HTMLIFrameElement,
  handler: FormSubmitHandler | MessageHandler
) {}

const myFrame = document.getElementsByTagName("iframe")[0]
        
const myForm = document.getElementsByTagName("form")[0]
        
handleMainEvent(myFrame, (val) => { // due to the function overloading typescript is capable of inferring and enforcing that the handler that goes with iframes is actually MessageHandler.
      
})
handleMainEvent(myForm, (val) => {
      
})
```

#### this keyword

Sometimes the `this` keyword is within a function that does not have its scope clear (at least for typescript). For instance, when a function is wired on the event of a DOM elements - onClick event of a button. In this case typescript is not capable of determining what exactly the keyword refers to and behaves like `any` - thus no useful autocomplete and properties.

```
<button onClick="myClickHandler">

...

function myClickHandler(event: Event) {
  this.disabled = true // typescript does no offer autocomplete and type safety
}

myClickHandler(new Event("click")) // this is a dangerous and misplaced call as it won't be within the expected scope and won't have any compilation error

```

To solve this issue and guide typescript that the `this` within the function will be related to the DOM element that emitted the event, it is necessary to either declare a `this type` with the functions arguments (example 1) or bind it (example 2).

```
function myClickHandler(this: HTMLButtonElement, event: Event) {
  this.disabled = true; // all properties from the button element are now available and we have type safety 
}

// Now we would have to bind the expected parameter with the function or use the call function to provide the correct this scope (or apply)

const myButton = document.getElementsByTagName("button")[0];
myClickHandler.call(myButton, new Event("click"))

const boundHandler = myClickHandler.bind(myButton)
boundHandler(new Event("click"))
```

*In other words, typescript does not know that the function was wired with a DOM element.

### Classes

#### Access Modifier Keywords

Typescript has only three access modifiers - private, public and protected. Their functionality are pretty much the same as in any other programming language - public means that everybody has access, private only the class itself and protected the class itself and its subclasses.

```
class Car {
  public make: string
  protected vinNumber: number
  private doorLockCode: string
  // the same applies to functions and methods

  constructor(make: string, ...) {
    this.make = make
  }

  protected unlockAllDoors() {
    ...
  }
}

// vanilla javascript introduced the concept of private field on version 3.8 and it is done with the hash character:
class Car {
  public make: string
  #doorLockCode: string
}
```
 
While the keyword `readonly` is not a access modifier per se as it has nothing to do with visibility, it can be used with class fields.
It is about (re)assignability than immutability - not allowing to point to something new rather than changing.

```
class Car {
  public readonly year: number;

  updateYear() {
    this.year++ // compilation error
  }
}
```

#### Param properties

It is a nice and convenient way of avoiding repetition (in the example above, each property was written 4 times: 1 for the field itself, 1 in the constructor args and 2 to assign the constructor parameter into the field).

```
class Car {
  constructor(public make: string, public model: string, ...) {

  }
}
```

#### Top and Bottom types

Types describe the values that a variable can store. 

Top types are types that describe anything thus are the most flexible ones. The set of things a top type could be is any value allowable in javascript. Two examples are `any` and `unknown`.

```
let flexible: any = 4
flexible = "An amazing text"
flexible = window.document

// the risks around using any stems from the fact that there will be no type safety of any sort from typescript. Leaving the errors to occur at runtime
let flexible: any = 4
flexible.typescript.wont.complain // typescript compiler is perfectly happy about it but the error will blow up at runtime when the code is being executed as it will clearly be a number
```

The essential difference between `any` and `unknown` is that the latter cannot be used unless it is narrowed via type guard.
Typescript forces you to verify before using it

```
let flexible: unknown = 4
flexible = "An amazing text" // reassign the value for another type as it does for `any`
flexible = window.document

flexible.typescript.will.complain.as.it.hasnt.been.narrowed // typescript won't allow it and will print a compilation error as it need to be type guarded first

if (typeof flexible === "string") {
flexible.substring(1)
}
```

Bottom types is the complete opposite of top types and describe things that can hold no possible values and is represented by the type `never`. This type is used along with exhaustive conditional where no final option is left behind:

```
class Car {
  drive() { }
}
class Truck {
  drive() { }
}
type Vehicle = Car | Truck

let myVehicle: Vehicle = obtainRandomVehicle()

if (myVehicle instanceof Car) { ... }
else if (myVehicle instanceof Truck) { ... }
else {
  const neverValye: never = myVehicle // there is no other option left, thus it is a never
}
```

The else clause can be considered dead code however it provides safety that is someone extends the Vehicle type, typescript will force them to handle it and not leave it behind as compilation error will light up every where that the previous narrowing is happening (the else case won't be never anymore).

```
class Boat() { } 
type Vehicle = Car | Truck | Boat

if (myVehicle instanceof Car) { ... }
else if (myVehicle instanceof Truck) { ... }
else {
  const neverValye: never = myVehicle // compilation error as now Boat is left.
}

```

To gracefully provide compilation error to the else case not having to resort to a void variable an UnreacheableError could be created:

```
class UnreacheableError extends Error {
  constructor(_nvr: never, message: message) {
    super(message)
  }
}

...
else {
  throw new UnreacheableError(myVehicle, `Unexpected vehicle type: ${myVehicle}`) // as soon as all the possibilities are not exhausted by the narrowing type guard, typescript will complain that myVehicle is not never anymore giving a nice compilation error.
}
```

#### Type guards and narrowing

When using union to define what values a variable could store causes typescript to not know what will be stored in that variable at runtime. In order to provide type safety, typescript only allows access to what is common to all potential objects and structures, if there is any - otherwise no access to anything.

To be able to access specific properties of potential types it is necessary to perform narrowing, that is a type guard check that ensures the type of object within a block or code (or branch) based on conditionals.

Some of the type guards provided by the api of the box are:

```
let value: Date | null | undefined | "pineapple" | [number] | {dateRange: [Date, Date]}

// instanceof
if (value instanceof Date) {
  value.getDate() // now it is ensured that it is a Date, not any of the other possibilities 
}

// typeof
else if (typeof value === "string") {
  value // not only does typescript know that is it a string but also that it is the string "pineapple"
}

// Specific value check
else if (value === null) {
  value // it can only be null in this case
}

// Truthy / falsy check - be careful with 0, empty string and false
else if (!value) {
  value // can only be undefined in this case
}

// Built-in functions
else if (Array.isArray(value)) {
  value.push(123) // at this point it is ensured to be an array
}

// Property presence check
else if ("dateRange" in value) {
  value // it can only be the last defined type
}

// never
else {
  // nothing else left - exhaustive conditional 
}

```

##### User defined type guards

User defined type guards are meant for way more comprehensive and complex cases, where using just the standard type guards would be cumbersome and at some extent impracticable.

Some times just creating conditionals (or even functions that return a boolean) based on the standard type guards is not enough to tell typescript which type it is coping with. In order to create a custom type guard it is necessary to use a especial return type that contains the `is` keyword and the type being tested.

```
interface Car {
  name: string
  make: string
  year: number
}

let maybeCar: unknown

// narrowing the value
if (isCarLike(maybeCar)) {
    maybeCar.make // now typescript is capable of understanding the type we are working with in this branch of code
}

// the guard
function isCarLike(valueToTest: any): valueToTest is Car { // especial return type "valueToTest is Car"
  return  (maybeCar &&
  typeof maybeCar === "object" &&
  "make" in maybeCar &&
  typeof maybeCar["make"] === "string" &&
  "model" in maybeCar &&
  typeof maybeCar["model"] === "string" &&
  "year" in maybeCar &&
  typeof maybeCar["year"] === "number")

}
```

Another way of creating custom type guards is with assertion. In case the provided type is not what is expected an error is thrown. The cool part of using this method is that it is not necessary to be used with conditional and a block of code.

```
// the guard
function assertsIsCarLike(
  valueToTest: any
): asserts valueToTest is CarLike {
  if (
    !(
      valueToTest &&
      typeof valueToTest === "object" &&
      "make" in valueToTest &&
      typeof valueToTest["make"] === "string" &&
      "model" in valueToTest &&
      typeof valueToTest["model"] === "string" &&
      "year" in valueToTest &&
      typeof valueToTest["year"] === "number"
    )
  )
    throw new Error(
      `Value does not appear to be a CarLike${valueToTest}`
    )
}

// using the guard
maybeCar // unknown
assertsIsCarLike(maybeCar)
maybeCar // CarLike
```

**** We need to be careful with user defined type guards to not end up with cases that are false positives and blow up in runtime - we are telling typescript what is fine but if we are not doing so concisely it will assume it is ok at compilation time but it won't work in runtime. 
Type guards are the glue between compile time validation and runtime behaviour.


#### Nullish (null, undefined and void)

Null indicates that there is a value for something and that value is nothing. Example:

```
const x = {
  email: 'abc',
  secondaryEmail: null // there is no secondary email.
}
```

Undefined generally means there's been a value defined yet. 

```
const formInProgress = {
  createdAt: new Date(),
  data: new FormData(),
  completedAt: undefined // this value hasn't been provided yet
}
```

Void should be used explicitily for function returns and it means that the return value of the function should be ignored.

##### Non-null assertion operator
The non-null assertion operator (!.) is used to cast away the possibility that a value might be `null` or `undefined`.

The value could still be null and this operator just tells typescript to ignore that possibility - in case the value is missing, the familiar error `cannot call foo on undefined` will be thrown.

```
type GroceryCart = {
  fruits?: { name: string; qty: number }[]
  vegetables?: { name: string; qty: number }[]
}

const cart: GroceryCart = {}

cart.fruits.push({ name: "kumkuat", qty: 1 }) // typescript says it is possibly null and gives a compilation error

cart.fruits!.push({ name: "kumkuat", qty: 1 }) // not anymore, but in this case the previously mentioned error will 
```

##### Definite assignment operator

Typscript enforces you to assign properties during initialization via constructors. If a property is not set, typescript show a compilation error. It happens that sometimes properties are not set directly on the constructor level but within a block of code and even though it is set synchronously typescript does not have a way of asserting it.

In order to tell typescript that you know the value will be available at construction time, it is necessary to use the `definite assignment operator (!:)`

```
class ThingWithAsyncSetup {
  setupPromise: Promise<any>
  isSetup!: boolean // definite assignment operator
  constructor() {
    this.setupPromise = new Promise((resolve) => {
      this.isSetup = false // this assignment will happen synchronous as it will be executed immediately when the promise is fired
      return this.doSetup()
    }).then(() => {
      this.isSetup = true
    })
  }

  private async doSetup() {
    // some async stuff
  }
}
```

### Generics

Generics are a way of creating types that are expressed in terms of other types. It leverage reuse not renouncing type safety, that is all that typescript is about.

Weak generic solution renouncing all type safety without generics - thing that we don't want to.
```
function listToDict(
  list: any[],
  idGen: (arg: any) => string
): { [k: string]: any } {
  const dict: { [k: string]: any } = {}

  list.forEach((element) => {
    const dictKey = idGen(element)
    dict[dictKey] = element
  })

  return dict
}

interface PhoneInfo {
  customerId: string
  phoneNumber: string
}

const phoneList: PhoneInfo[] = [{customerId: 'asd', phoneNumber: 'asdasd'}, {customerId: 'sad', phoneNumber: 'sadasd'}]
const dict = listToDict(phoneList, (p) => p.customerId)
dict.this.does.not.exist.and.will.blow.up.at.runtime;
```

Generic solution with type safety enforced:
```
function listToDict<T>(
  list: T[],
  idGen: (arg: T) => string
): { [k: string]: T } {
  const dict: { [k: string]: T } = {} // return a dictionary with arbitrary keys (index signature) that store the generic type T

  list.forEach((element) => {
    const dictKey = idGen(element)
    dict[dictKey] = element
  })

  return dict
}

interface PhoneInfo {
  customerId: string
  phoneNumber: string
}

const phoneList: PhoneInfo[] = [{customerId: 'asd', phoneNumber: 'asdasd'}, {customerId: 'sad', phoneNumber: 'sadasd'}]
const dict = listToDict<PhoneInfo>(phoneList, (p) => p.customerId)
// even if the types weren't explicitly define in any of the two previous lines, typescript would be able to stick at least to the type structure (an array of arbitrary keys that store types that contain customerId and phoneNumber). 

dict.key.{customerId, phoneNumber} // typescript can still provide type safety because it nows the return object will be a dictionary of arbitrary key and that each key will store a PhoneInfo. In this case it would only be necessary to ensure that the key is present.

dict.key.somethingElse // it would case compilation error.
```

##### Generic constraints

Generic constraints allow us to describe the `minimum requirement` for a type param. It enables hight level of flexibility while still providing minimal structure and behaviour:

```
interface HasId {
  id: string
}

interface Dict<T> {
  [k: string]: T
}

// similar to the previous example but enforces that the provided object has at least an id. It will still return expected complete object

function listToDict<T extends HasId>(
  list: T[]
): Dict<T>  {
  const dict: { [k: string]: T } = {} // return a dictionary with arbitrary keys (index signature) that store the generic type T

  list.forEach((element) => {
    const dictKey = element.id
    dict[dictKey] = element
  })

  return dict
}

const obj = {
  id: "a",
  abc: "asdd"
}

const result = listToDict([obj]);
result.a.id // a is the key of the dictionary
result.a.abc
result.a.def // compilataion error as the provided object does not have a def property. 
```


#### Casting

Generally to cast from one type to another, the keyword `as` is used:

```
const sameAs = window as any as number
```

However it is possible to achieve the same with functions and it could be dangerous (aka convenience cast):

```
function returnAs<T>(arg: any): T {
  return arg // casts to T and it is dangerous and typescript is not able to help much here as arg could be anything, include someting that is castable to T. 
}

const first = returnAs<number>(window) // no compilation error but will certainly blow up at runtime
```

* Don't make things generic unless there's real value. Premature abstraction is bad and makes it harder for you and others understand your code. *

##### Notes

Type systems:
static vs dynamic type systems
nominal vs structural types

Duck typing = if it looks like a duck, swims like a duck and quacks like a duck it is probably a duck - another way of describing dynamic typing.

Strong type and Weak type are terms generally used to refer to static and dynamic types but there is no concensus to what it really means.