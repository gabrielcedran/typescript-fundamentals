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

##### Notes

Type systems:
static vs dynamic type systems
nominal vs structural types

Duck typing = if it looks like a duck, swims like a duck and quacks like a duck it is probably a duck - another way of describing dynamic typing.

Strong type and Weak type are terms generally used to refer to static and dynamic types but there is no concensus to what it really means.