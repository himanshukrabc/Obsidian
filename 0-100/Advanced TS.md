## Pick
```TS
interface User{
    id:string;
    name:string;
    age:number;
    email:string;
    password:string;
}

//Suppose you need to update just name age and password.
// You can do the following things
//This will lead to large number of functional arguments as program grows.
function updateUser(name:string,age:number,password:string){}

//This approach will ask you to update any interface changes in two places.
// This can be overlooked and is fatal in nature if not fixed.
interface updateProps {
    name:string;
    age:number;
    password:string;
}
function updateUser1(prop:updateProps){}

type updProps = Pick<User,'name'|'age'|'email'>
function updateProps2(prop:updProps){}
```

## Partial
- Helps in making some properties optional from a given interface
- This may happen in updates where you are not sure of which properties the user wants to update.
```ts 
interface updPropsOpt = Partial<updateProps>
```

## Read-Only
- Even when we declare a array/object with const, the values stored in them gets updated.
- This is because node just checks the reference and not the value.
- ReadOnly helps implement this. 
- Usually used for storing config.
```ts
type readOnlyUser = Readonly<User>;
type User2{
	readonly name:string;
	readonly age:number;
}
```

## Defining Type for Objects, Records
```ts
type Users{
	[key:string]:{name:string,age:number}
}

Users u={
	"emp1":{
		name:"HK",
		age:21
	},
	"emp2":{
		name:"Himanshu",
		age:23
	}
}
```
- Simpler way to define types for objects
```ts
type User{
	name:string,
	age:number
}
type Users = Record<string,User>
```

## Maps
```ts
const users = new Map<string,User>();
users.set('emp1',{name:"hK",age:12});
users.get('emp1');
users.delete('emp1'); 
```

## Exclude
- Used in Enums
```ts
type EventType = 'click'|'scroll'|'mouseover'
type ExculdedEventType = Exclude<EventType,'scroll'>
```

## Type inference in [Zod](Express%20and%20Backend#Zod)


```ts
const schema = z.object({
    username:z.string().min(5, { message: "Must be 5 or more characters long" }),
    email:z.string().email(),
    dob1:z.optional(z.string().date()),
    dob2:z.string().date().optional(),
    id:z.number(),
    country:z.literal("IN").or(z.literal("US")),
    
});
type User = z.infer<typeof schema>;
```
