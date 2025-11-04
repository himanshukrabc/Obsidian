## Strongly typed vs Loosely typed languages
- Strongly typed languages require variables to have a type defined. It can only store just that type of values;
- Loosely typed values dont require a data type declaration and can be used to store multiple types of values one after another.
```js
let x=10;
x="sdf";
```

###### - typescript just adds types to javascript
## How does typeScript run?
- TS does not run on the browser. It is compiled down to typescript.
- If any error occurs then the transpilation to JS fails.
## Creating a TS project
```
npm init
npx tsc --init
```
- tsconfig.json => TS config file. Contains ts configurations.

## Running the TS file
```
tsc -b //will build the current file. Creates a JS file.
node file_name.js
```

## Declaring Variables
```TS
const x:string = "Hello"
const y:number = 12
const z:boolean = true
const a:null;
const a:undefined;
const a:any; // lets you store any type of value.
```
- Even when you want to store different types of data you need to declare the variable with any type.
## Declaring Functions
```TS
function sum(num1:number,num2:number):number{
    return num1+num2;
}
```
- you need to assign a return type to a function.
- Although TS can **infer** the type based on the function. So can be neglected.

## Functions as Arguments
```ts
function runAfter1s(fn:()=>void){
    setTimeout(fn,1000);
}

runAfter1s(()=>{console.log("Heyyyy")});
```

## tsconfig.ts
- **target** => The ecmascript version that TS need to be compiled to.
- **rootDir, outDir** => keep the TS and JS files seperate by specifying the path. This way you can init a .gitignore to remove all compiled JS files.
- **noImplicitAny : true,** => If true it does not allow any datatype vars.
- **removeComments : true** => Removes all the comments in the compiled JS file.

## Interfaces
- Used to assign types to objects.
- use ? to assign optional fields
```js
interface User{
    name:string,
    age:number,
    username?:string // Optional Argument
}

let me:User = {
    name :"Himanshu",
    age:22,
    username:"himanshukrabc",
};
let you:User = {
    name :"Himanshu",
    age:22
};
```
- Interfaces can also be implemented.
#### Implementing Interfaces
```js
interface Person{
	name:string,
	age:number,
	greet(phrase:string):void
}

class Employee implements Person{
	name:string;
	age:number;
	constructor(n:string, a:number){
        this.name=n;
        this.age=a;
    }
    greet(phrase:string){
        console.log("Hello  "+phrase);
    }
}
```

## Types
- Similar to interfaces but cannot be implemented.
```TS
type User{
	name:string,
	age:number
}
```
#### Additional Features
1. **Unions** 
```ts
type numberOrString = number | string;
function greet(phrase:numberOrString){
	console.log(phrase);
}
greet(1);
greet("1");
```
2. **Intersections**
```TS
type Emp={
    name:string,
    dep:string
};
interface Person{
    name:string,
    age:number
};

type Manager=Emp&Person;
type Manager1={
    name:string,
    dep:string,
    age:number
}
```
## Arrays
```TS
const arr:number []=[1,2,3,4]
function getMax(arr:number[]){
    var mx:number=arr[0];
    for(var i:number=0;i<arr.length;i++){
        if(arr[i]>mx)mx=arr[i];
    }
    return mx;
}
```

## Enums
```TS
type keyInp = "Up"|"Down"|"Left"|"Right"
function keyPressed1(key:keyInp){
}
keyPressed1("North");//Error
enum Direction{Up,Down,Left,Right};
function keyPressed2(key:Direction){
}
keyPressed2(Direction.North);//Error
console.log(Direction.Up);//0
console.log(Direction.Down);//1
console.log(Direction.Left);//3
console.log(Direction.Right);//4
enum Direction1{Up="Up",Down="Down",Left="Left",Right="Right"};
enum Direction{Up=10,Down=23,Left,Right}; //Left gets 24, Right 25
```
- Enum gives suggestions.

## Generics
- Declaring functions of general types.
```TS
function identity<T> (arr:T[]):T{
	return arr[0];
}
var val = identity(["HK","HHK"]);

```

## Import/Export\
- follows es6 methodology
```TS
import express from "express";
export const a=1;
export const b=1;
export default const c=1;

// ---------------------------- //
import c,{a,b} from "./file.ts"
```


## [[Advanced TS]]