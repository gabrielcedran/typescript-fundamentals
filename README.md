# Typescript Fundamentals

## Bootstrapping the simplest JS project only with Typescript

1. Initialise the project with the command `npm init` (or create the package.json manually)
2. Add typescript as a development dependency (`npm install typescript -D`)
3. Create npm script to compile typescript into vanilla JS. It will speed up development (and convenience). 
In the package.json `scripts` property declare the new script name as `dev` with the value `tsc` (`"dev": "tsc --watch --preserveWatchOutput"`). To execute run `npm run dev`.
To run inline without the npm script, simply use the `npx` command with the same parameters (`tsc --watch --preserveWatchOutput`)
4. Create the typescript configuration file (tsconfig.json) with the command `tsc --init` (`npx tsc --init`).
The most important attributes, at least at first, are `compilerOptions.outDir`, `compilerOptions.target` and `include`.
    1. If the project is meant to be run with node, add the `compilerOptions.module` with the value `commonjs`. To run the generated source simply execute `node {file}.js`.
5. Enable tsc to generate type definition if the project is meant to be a library so that other modules can make use of the types. Properties `declaration` and `declarationMap` in the tsconfig.json