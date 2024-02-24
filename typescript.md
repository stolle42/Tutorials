# What is Typescript
Typescript is a superset of Javascript. It extends Javascript by adding a type system that can help the programmer to spot errors early. There is a typescript compiler (TSC) that can convert typescript into javascript, which can then be run on websites
# Prerequisites
- Javascript
# Tooling and preparation
Install the following tools:
- ts-node
- tsc
- Node.js
# Hello World
hello world in typescript will be similar to the one in javascript, except with type annotations:
```ts
let myString: string = 'Hello World!';
console.log(myString);
```
Save the file with a name (e.g. main.ts), then run the compiler:
```
ts-node main.ts
```
The console should now output `hello world!`

Actually, ts-node does not run the code directly. Instead, it generates a JavaScript-Code, which it then executes by Node.js. If you want to save the generated JavaScript-Code in a file, run

```
tsc main.ts
```
It should generate the file main.js, which you can then run with the Node.js compiler, or integrate it into your website (see the JavaScript tutorial for more elaboration)