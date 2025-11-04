## Theory
ECMA Script => Provides a set of instructions for writing scripting lang compilers
Javascrpt => Implementation of ECMA Script.
V8 Engine - It is a compiler written on browser to compile JS

NodeJS =>V8 compiler was taken out of browser and added some extra features. It is a runtiime that makes JS runs on servers.

Bun => Competitor of NodeJs. It is significantly faster. Written in zig

NodeJS -> create CLIs, VideoPlayers, HTTP Servers.

HTTP Server - 
- HTTP is a protocol that helps machine communicate.
- It is some code that follows HTTP and can communicate with clients.
![[Pasted image 20240508135743.png]]

What happens when you send a request?
1. Parse the url
2. DNS lookup/resolution => convert domain to IP

Referer Header => checks what domain is trying to access the IP. Multiple domains can point to the same server/IP. The server then decides what response to send based on the domain.

## Middlewares
- If you require a logic to work on all the routes before they process it, you can define functions to do so. Such functions are called middlewares.
```javascript
function middleware1(req, res, next){}
function middleware2(req, res, next){}

app.get("/health",middleware1, middleware2, ..., function(req,res)=>{
	res.status(200).send({});
})

app.get("/health2",middleware1,..., function(req,res)=>{
	res.status(200).send({});
})

```
#### How do middlewares work?
- The app.get(cb1,cb2,..) can take multiple callback functions.
- Control flow => cb1->cb2->cb3....
- Each cb(req,res,next) has in it a parameter next. It is a function that calls the next function in line.
- The final function doesn't have this. It sends res.
- the next() is not the next callback function. It is an internal function that calls the next callback function with the correct params.
```javascript

app.get("/health",(req,res,next)=>{
	//logic
	next();
},(req,res)=>{
	//logic
	res.send();
});
```
- Rate limiting is done using middlewares.
- ##### app.use(middleware1)
	- This lets the middleware to be called in every route after this line.
- Do input validation always.

## Global Catches
- If any error comes in any route it handles the task of displaying a generic response so that the user is not displayed the error stack.
- This protects the server.
- These are Error handling middlewares.
- This needs to be **declared at the end** of all routes for which it catches the error.
```javascript

app.use((err, req, res, next())=>{
	res.json({
		msg:"error on server"
	});
})
```

## Zod
- Schema validator
- define schema for each endpoint and validate the request body
- There is also support for editing schemas and picking out properties.
- Chectout the [docs](https://zod.dev/)
- schema.safeparse() => validate
```javascript

const express = require("express");
const z = require("zod");
const app = express();

app.use(express.json());

const schema1 = z.string();
const schema = z.object({
    username:z.string().min(5, { message: "Must be 5 or more characters long" }),
    email:z.string().email(),
    dob1:z.optional(z.string().date()),
    dob2:z.string().date().optional(),
    id:z.number(),
    country:z.literal("IN").or(z.literal("US")),
    
});

app.post("/health-checkup", function(req, res) {
    const kidneys = schema.safeparse(req.body);
    const kidneyLength = kidneys.length;
    res.send("you have " + kidneyLength + " kidneys");
});

const port = 3000;
app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});


```


## [[Advanced TS#Type inference in Zod]]
## Fetch API
```js
function getData(){
	fetch(_url).then((res)=>{
		res.json().then((res)=>{
			console.log(res);
		})
	})
}
async function getData(){
	const response = await fetch(_url)
	const data = await response.json()
	console.log(data)
}

```

## Authentication
##### 1. Hashing
	Store the data in hashed state
##### 2. Encryption
	When communicating the data is sent in a hashed manner. Only the person 
	has the key will be able to decrypt and make sense of the data.
##### 3. JWT
- It takes JSON data as input and converts it into a token
- Token -> Just a string
- anyone with a Token can convert to the JSON
- When creating the token, you also pass a secret/password. This password needs to be passed with the token to verify the JSON.
- Decoded by anyone but verified by using password only.
- Instead of checking on DB if the user is authentic, you just verify the token.
- This saves a DB call.
##### 4. Local Storage
- This the place where the Token is stored.
- Generally, the backend sends a userdata in JWT format. This is stored in the local storage. When any request is made the JWT is sent. This is then used to verify the data.
```js

const express = require("express");
const jwt = require("jsonwebtoken");
const app = express();

app.use(express.json());

const users=[
    {username:"qwe1",password:123},
    {username:"qwe2",password:1123},
    {username:"qwe3",password:2123}
]

const userExists = (username,password)=>{
    ret = false;
    users.forEach((user)=>{
        if(user.username==username&&user.password==password)ret=true;
    })
    return ret;
}

app.post("/signin", function(req, res) {
    console.log("afipugfaipud")
    console.log(userExists(req.body.username,req.body.password));
    if(!userExists(req.body.username,req.body.password)){
        res.status(403).json({
            msg:"No User found"
        });
    }
    var token=jwt.sign({username:req.username},"BE_Pass");
    return res.status(200).json({
        token:token
    });
});

app.get('/getusers',function(req,res){
    const token=req.headers.auth;
    try{
        const decoded=jwt.verify(token,"BE_Pass");
        const username=decoded.username;
        res.status(200).json({users:users.filter((user)=>{return user.username!=username})});
    } catch(err){
        return res.status(403).json({
            msg:"invalid token"
        });
    }
})

const port = 3000;
app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});

```


## CORS
- To set which urls are allowed to use the backend server.