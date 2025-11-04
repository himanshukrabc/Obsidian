## Server Based Architecture
- Cloud Providers like AWS provide VMs running on their data centers which can be used to deploy apps to the internet.
- The apps are placed in auto scaling groups and deployed on a kubernetes cluster.
- There are downsides to this -
	- Taking care of when and how to scale
	- Base cost even if no one is visiting the site
	- Monitoring various servers to make sure no server is down.
## Serverless Architecture
- The VM is charged on a per request basis.
- The Cloud provider dynamically manages the allocation and provisioning of servers.
- Deployment, AutoScaling is automatically taken care for.
- Downsides -
	- Cold Start Problem - If the server is down(due to no visits) and a user visits, the server will take a lot of time to process request.
- Providers - AWS lamda, Google Cloud Functions, Cloudflare Workers.

## Advantages over AWS Lambda
- Does not use nodeJs runtime and is faster than Node. Under the hood it still uses V8 engine.
- **Isolates** - Instead of creating node processes for each worker, there are several isolates running inside a single process.

## Creating cloudflare backend locally
```
npm create cloudflare -- app-name
```

## Wrangler
- This is the CLI of Cloudflare. This helps in deploying your codes, starting and creating the servers.
#### Pushing Code through wrangler
```
npx wrangler login
npm run deploy
```
```
// get user data
npx wrangler whoami
```
```
// To deploy another worker
Change the app name in wrangler.toml file
```
## Cloudflare Code
- default code without library
```JS
export default {
	async fetch(request:Request, env:Env, ctx:ExecutionContext): Promise<Response> {
		console.log(request.body);
		console.log(request.headers);
		// YOu can get the uri from request
		const uri=request.url.replace(/^https:\/\/.*?\//gi,"/");
		if(request.method == "GET"){
			console.log("------------------");
			console.log(uri);
			if(uri=="/user")return new Response("User Get");
			else return new Response("New message received");
		}
		else if(request.method == "POST"){
			return Response.json({msg:"New message received"});
		}
		return Response.json({msg:'Hello World!'});
	},
} satisfies ExportedHandler<Env>;

```

## Moving Express/Node backend to Cloudflare Workers
- Convert your code as generic as possible(functions)
![[Pasted image 20240726185858.png]]
- Rounting must be done through cloudflare and you call the functions internally.
## Using honoJs
- This is a routing library like express that works on cloudflare and other serverless providers.
- This is very similar to Express.
- Creating a new file - 
```
npm create hono@latest app-name
```

```ts
import { Hono } from 'hono'

const app = new Hono()

// middleware
app.use(async(c,next)=>{
  if(c.req.header("Authorization")){
    await next();
  }
  else return c.text("Auth headrer not found");
})

app.post('/', async (c) => {
  const body = await c.req.json();
  console.log(body);
  //get Authroisation header
  console.log(c.req.header("Authorization"));
  // get param named query parameter
  console.log(c.req.query("param"));
  return c.text('Hello Hono!');
});

export default app
```

