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

