Whenever a component rerenders, the global variables are not changed.
## Debouncing
- When rendering objects as the user types, You need to wait for a few milliseconds after the user stops typing and then show the result.
```Js

const timeout;
function debouncePopulateDiv(){
	clearTimeout(timeout);
	timeout=setTimeout(populateDiv,100);
}  

async function populateDiv(){
	const a=document.getElementById('a')
	const b=document.getElementById('b')
	try{
		const resp=await fetch(`http://get-sum.com?a=${a}&b=${b}`)
		return resp;
	}catch(err){
		return err;
	}
}
```

## Shortcomings of HTML
##### Dynamic Websites has very bulky code
- Simple todo app has this much code.
```HTML
<html lang="en">
<body>
    <input type="text" id="title"><br><br>
    <input type="text" id="desc"><br><br>
    <button onclick="addTask()">Add Task</button><br><br>
    <div id="tasks"></div>
</body>
<script>
    function markDone(id){
        alert("Good Job!");
    }
    function createTask(title, desc, id){
        console.log("Yay!!!")
        let firstGrandChild = document.createElement('div')
        firstGrandChild.innerHTML=title
        let secondGrandChild = document.createElement('div')
        secondGrandChild.innerHTML=desc
        let thirdGrandChild = document.createElement('button')
        thirdGrandChild.innerHTML='Done!'
        thirdGrandChild.setAttribute('onclick',`markDone(${id})`)
        let child =document.createElement('div')
        child.setAttribute('id',id)
        child.appendChild(firstGrandChild)
        child.appendChild(secondGrandChild)
        child.appendChild(thirdGrandChild)
        return child
    }
    let globalId=1;
    function addTask(){
        let title = document.getElementById('title').value;
        let desc = document.getElementById('desc').value;
        let div = document.getElementById('tasks');
        /*
        let innerHTML = div.innerHTML;
        div.innerHTML = innerHTML + 
        `<div>
        <div>${title}</div>
        <div>${desc}</div>
        <button>Mark as done</button>
        </div>`
        */
        div.appendChild(createTask(title,desc,globalId++))
    }
</script>
</html>

```
## Virtual DOM
##### Working with states is difficult => **No Virtual DOM**
- When states are involved, the website needs to query the backend on regular intervals to check if the data has changed.
- Also everytime the backend will return the entire state and not what has changed. 
- This leads to reload of the entire DOM.
```js
window.setInterval(async function() {
    const res = await fetch("[7](https://sum-server.100xdevs.com/todos)");
    const json = await res.json();
    updateDemoAppState(json.todos);
}, 5000);
```
- Better way of doing this is to change only the differences. Using something called the **Virtual DOM**
- This is done by using some different calc algo.
- What does Virtual DOM do?
	1. Update of state variable
	2. Figuring out the updated and deleted nodes
	3. Updating and Deleting nodes => ReactDOM/ReactNative
- The piece of code that does this is called Reconciler.
#### Reconcilation
It is the process by which react updates the DOM to match the virtualDOM. It consists of the above 3 steps.

## Getting Data from Backend
- Using fetch like this will lead to infinite requests to the backend.
```jsx
function App() {
  const [todos, setTodos] = useState([]);

  fetch("http://localhost:3000/todos")
    .then(async (res) => {
      const json = await res.json();
      setTodos(json.todos);
    });

  return (
    <div>
      <CreateTodo/>
      <Todos/>
    </div>
  );
}
```
- This is because when you fetch todo, the setState causes the state to change leading to rerender of component and another fetch req.
- Therefore we use useEffect Hook.
  
## How to get data from input elements in React
```js
const [title, setTitle] = useState("");

return <div>
  <input id="title" type="text" placeholder="title" onChange={function(e) {
    const value = e.target.value;
    setTitle(e.target.value);
  }}/><br />
</div>

```
- However this leads to multiple rerenders leading to unoptimized websites.
## Why does react return single component/tag?
- Makes the reconcilation process easy.
- React returns XML.
- If you want to return multiple elements, add it in a react.fragment **< ></ >**
## Re-rendering/Memoization => React.memo()
- Whenever state changes the components influenced by it rerender.
- All the components within it will also rerender.
- **NOTE:** If html is there it will not rerender.
- In the below code cchanging the title rerenders both Headers and not the JS.
```js
import React, { Fragment } from "react";
import { useState } from "react";

function App() {
  const [title, setTitle] = useState("my name is harkirat");

  function updateTitle() {
    setTitle("my name is " + Math.random());
  }

  return (
    <Fragment>
      <button onClick={updateTitle}>Update the title</button>
      <Header title={title} />
      <Header title="harkirat2" />
      Hello There!!!
    </Fragment>
  );
}

function Header({ title }) {
  return <div>{title}</div>;
}

export default App;
```
- Rerender can happen in 2 cases
	- If state changes
	- If a parent rerenders.
- You need to minimize the number of rerenders.
- Solutions - 
	- If possible push the state down. => Only rerender the one whose state changed. => Reduced parent rerendering.
	- user React.memo => makes the component rerender only when the props to it have changed.
- Why not always memoize => overhead increases due to comparis
```js
import React, { Fragment } from "react";
import { useState } from "react";

function App() {
  const [title, setTitle] = useState("my name is harkirat");

  function updateTitle() {
    setTitle("my name is " + Math.random());
  }

  return (
    <Fragment>
      <button onClick={updateTitle}>Update the title</button>
      <Header title={title} />
      <Header title="harkirat2" />
      <Header title="harkirat2" />
      <Header title="harkirat2" />
      <Header title="harkirat2" />
      Hello There!!!
    </Fragment>
  );
}

function Header= React.memo(({ title }) =>{
  return <div>{title}</div>;
})

export default App;
```


## Ids in react
- Whenever creating multiple components using map()/etc, you need to pass ids.
- Each Component in a list/array of components must have a key prop.
- This helps react keep track of individual components of the list, if any change is made to them.
- This way it will not rerrender the entire list and only the components which changed
## Wrapper components
- These are components which take another component as props.
- Generally used for styling purposes.
```jsx
function App() {
  return <div>
    <CardWrapper innerComponent={TextComponent} />
    <CaedWrapper2>
		<TextComponent></TextComponent>
    </CaedWrapper2>
  </div>
}

function TextComponent() {
  return <div>
    hi there
  </div>
}

function CardWrapper({ innerComponent }) {
  return <div style={{ border: "2px solid black" }}>
    {innerComponent}
  </div>
}

function CardWrapper2({ children }) {
  return <div style={{ border: "2px solid black" }}>
    {children}
  </div>
}
```
## Side Effects
- Any piece of code that is not involved with components/state.
- Eg- fetching data from backend, setTimeout, setInterval etc.
- These are actions that effect other components and cannot be performed during rendering as they cause infinite renders.
## Hooks
- These are functions that let you hook into the lifecycle features.
- **Lifecycle of Components** - 
	1. Mount - It means the time the component is added-to-DOM/re-rendered. => function that you write inside the useeffect
	2. Unmount - It means when the state changes and component is removed for re-rendering. => function that you return from the useEffect
- so basically hooks help us in rerenders - 
	1. **useState** => define cause of rerenders(state variables)
	2. **useEffect** => Do a Effect/side Task on rerender
	3. **useMemo** => Persist value from calculation logic across rerender
	4. **useCallback** => persist a function across rerenders. This helps in not rerendering a child which has a function as a prop.
	5. **useRef** => Helps in getting dom elements, storing numbers/values across rerenders.(counting rerenders)
	6. **memo** => persist a component across rerenders
## useState
- Used to define the state of the component.
- Whenever the state updates, the component it was defined in and all its children rerender.
```jsx
let [value,setValue] = useState(0/*Default Value*/);
```
```jsx
import { useState,useCallback,memo } from "react";

export function Assignment1() {
    const [count, setCount] = useState(0);
	// Handling setcount this way frees it from the use of count.
	// If done the normal way(setCount(count+1)) the function will again be redefined on rerender of the compponent =>. rerender of the counter buttons 
	const handleIncrement=useCallback(()=>{
        setCount((curCount)=>{return curCount+1});
    },[]);

    const handleDecrement=useCallback(()=>{
        setCount((curCount)=>{return curCount-1});
    },[]);
    
    return (
        <div>
            <p>Count: {count}</p>
            <CounterButtons onIncrement={handleIncrement} onDecrement={handleDecrement} />
        </div>
    );
};

const CounterButtons = memo(({ onIncrement, onDecrement }) => {
    return  <div>
        <button onClick={onIncrement}>Increment</button>
        <button onClick={onDecrement}>Decrement</button>
    </div>
});

```
## useEffect - 
- Runs the callback when the component mounts/dependency array changes.
- It allows to perform Side Effects in functional components.
- performs componentDidMount(), componentDidUpdate(), componentWillUnmount()
```jsx
useEffect(function(){}/*Callback*/,[]/*Dependency Array*/)
useEffect(function(){}/*Callback*/,[]/*Dependency Array*/)
```
- The dependency array allows to provide some conditions(-> state variables) when the callback needs to run.
- Problem in useEffect with async calls => If once dependency changed and while the server was processing again the dependency changed. => will lead to mapping of the 1st request on second dependency change.
```jsx
useEffect(function()=>{
	async function getData(){
		let newVal = await axios.get("_url_");
		setVal(newVal);
	}	  
	getData()
},[state_var]) 
```
- Usually useEffect returns a function. This is called the componentUnmount function that runs when the component unmounts.
- Remember => this will not run when state changes. Only when the component where the useEffect was introduced unmounts.
```jsx
useEffect(function()=>{
	async function getData(){
		let newVal = await axios.get("_url_");
		setVal(newVal);
	}	  
	getData()
	return ()=>{console.log("Component Unmounted");}
},[state_var])
```
## useMemo - 
- It is used to memoize values across rerenders.
- If a expensive logic is used to compute a value, and the rerender has nothing to do with that value, it does not make sense to recalculate the value again.
- Here the for loop will be called everytime the count changes.
```jsx
function App() {
  const [count, setCount] = useState(0);
  const [inp, setInp] = useState(0);

  function updateCount() {
    setCount(count+1);
  }

  let sum=0;
  for(let i=0;i<inp;i++){
    sum+=i+1;
  }
  console.log("Called");

// Sollution 1 => Will cause extra rerender. 1st due to the counter change and then due the use Effect
const [displayVal, setDisplayVal]=useState(0);
useEffect(()=>{
  let sum=0;
  for(let i=0;i<inp;i++){
    sum+=i+1;
  }
  setDisplayVal(sum);
  console.log("Called - 2");
},[inp])

// Solution 2 => 
let sum = useMemo(()=>{
  let sum=0;
  for(let i=0;i<inp;i++){
    sum+=i+1;
  }
  console.log("Called - 3");
  return sum;
},[inp]);

  return (
    <Fragment>
      <input type="text" onChange={(e)=>{setInp(parseInt(e.target.value));}
      }></input>
      <br></br>
      sum is {sum}
      sum is {displayVal}
      <br></br>
      <button onClick={updateCount}>Count ({count})</button>   
    </Fragment>
  );
}
```

## useCallback - 
- used to memoize functions.
- Suppose a state variable changes which does not affect our component.
- If you pass a variable to said component, even after rerender when a new variable is defined, the values will remain the same and not cause the component to rerender.
- However if we are passing a function into component, the new function generated after rerender, will not be the same as the old function. => Rerender of said component
```jsx
import { memo, useEffect, useState, useCallback } from "react";

function App() {
  var a = 10;
  const [counter, setCounter] = useState(0);

  var b=function(){
    console.log("Hey");
  }

  var c = useCallback(()=>{console.log("asdiuf")},[]);

  return (
    <div>
      <button onClick={() => setCounter(counter + 1)}>Click me! ({counter})</button>
      <DemoFun a={b} />
      <DemoFun a={c} />
      <DemoVar a={a} />
    </div>
  );
}

const DemoFun = memo(function({a}) {
  console.log("render", a);
  return <div>Hi there</div>;
});

const DemoVar = memo(function({a}) {
  console.log("render", a);
  return <div>V Hi there</div>;
});

export default App;
```
- here when counter changes demoFun will be rerendered every time.
- useCallback() allows to make functions persist across rerenders.

## useRef
- Used to gain access to DOM elements.
```jsx
import React, { useEffect, useRef, useState } from 'react';

function App() {
  const [incomeTax, setIncomeTax] = useState('20000');
  const divRef = useRef();

  useEffect(() => {
    setTimeout(() => {
      divRef.current.innerHTML = 10;
      console.log(divRef.current.innerHTML);
    }, []);
  });

  return (
    <div>
      hi there, your income tax returns are due <div ref={divRef}>incomeTax</div>
    </div>
  );
}

export default App;
```
- Used to create create variables whose value persists across rerenders.
## Custom Hooks
- When you use a bunch of hooks and dont want to display the logic in the current file.
- Define a custom hook that returns the value it calculates.
- It is a piece of code that is using a set of hooks.
- You can also add a loading variable to return if the request through hooks has been completed or not.
```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function useTodos(){
  const [todos, setTodos] = useState([]);
  const [loading,setLoading] = useState(true);
  useEffect(() => {
    axios.get("https://sum-server.100xdevs.com/todos")
      .then(res => {
        setTodos(res.data.todos);   
        setLoading(false);
      });
  }, []);

  return {todos,loading};
}

function App() {
  const {todos,loading} = useTodos();
  if(loading)return <div>Loading....</div>
  return (
    <div>
      {todos.map(todo => <Track todo={todo}/>
      )}
    </div>
  );
}

function Track({todo}){
  return<div key={todo.id}>
  <h3>{todo.title}</h3>
  <p>{todo.description}</p>
  <br />
</div>
}

export default App;
```
- Auto RePolling hook => hook that polls the backend every few seconds to get new data.
```jsx
function useTodos(n){
  const [todos, setTodos] = useState([]);
  const [loading,setLoading] = useState(true);
  
  useEffect(() => {
    const value=setInterval(() => {
      axios.get("https://sum-server.100xdevs.com/todos")
        .then(res => {
          setTodos(res.data.todos);   
          setLoading(false);
        });    
    }, n*1000);
    axios.get("https://sum-server.100xdevs.com/todos")
    .then(res => {
      setTodos(res.data.todos);   
      setLoading(false);
    });    
    return ()=>clearInterval(value);// If we dont clear the timeout, whenever the n changes, new clock will start while the old one will still continue. Generating multiple sets of intervals.
  }, [n]);
  return {todos,loading};
}
```
```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function useTodos(n){
  const [todos, setTodos] = useState([]);
  const [loading,setLoading] = useState(true);
  
  useEffect(() => {
    const value=setInterval(() => {
      axios.get("https://sum-server.100xdevs.com/todos")
        .then(res => {
          setTodos(res.data.todos);   
          setLoading(false);
        });    
    }, n*1000);
    axios.get("https://sum-server.100xdevs.com/todos")
    .then(res => {
      setTodos(res.data.todos);   
      setLoading(false);
    });    
    return ()=>clearInterval(value);
  }, [n]);
  return {todos,loading};
}

function App() {
  // const {todos,loading} = useTodos(5);
  const online = useIsOnline();
  const {x,y} = usePosition();
  // if(loading)return <div>Loading....</div>
  return (
    <div>
      {/* {todos.map(todo => <Track todo={todo}/>
      )} */}
      You are {online?"Online":"Offline"}
      x:{x}, y:{y}
      <SearchBar/>
  </div>
);
}

function Track({todo}){
  return<div key={todo.id}>
  <h3>{todo.title}</h3>
  <p>{todo.description}</p>
  <br />
</div>
}

const SearchBar = () => {
  const [inputValue, setInputValue] = useState('');
  const debouncedValue = useDebounde(inputValue,500);
  return (
    <>
    <input
      type="text"
      onChange={(e) => setInputValue(e.target.value)}
      placeholder="Search..."
    />
    {debouncedValue}
    </>
  );
}

function useDebounde(value, time){
  // const timer=useInterval(time);
  const [debVal,setDebVal] = useState(value);
  const timeout = setTimeout(()=>{
    setDebVal(value);
  },time);
  useEffect(()=>{
    clearTimeout(timeout);
  },[value]);
  return debVal;
}

function useIsOnline(){
  const [online,setOnline] = useState(window.navigator.onLine);
  useEffect(()=>{
    window.addEventListener("online",()=>{setOnline(true)});
    window.addEventListener("offline",()=>{setOnline(false)});
    return ()=>{
      window.removeEventListener("online",()=>{setOnline(true)});
      window.removeEventListener("offline",()=>{setOnline(false)});    
    }
  },[])
  return online;
}

function usePosition(){
  const [pos,setPos] = useState({x:0,y:0});
  useEffect(()=>{
    window.addEventListener("mousemove",(e)=>{
      setPos({x:e.screenX,y:e.screenY});
    })
  },[])
  return pos;
}



export default App;

```
## Routing, Lazy Loading
#### Single Page Applications
- Pre-React, every time you go to a different page, it asks the server for new html/css/js. => Hard Reload
- In React, all the pages related to a website are sent in a single package called **Client Side Bundle**.
#### Client Side Routing
- How do you show different pages based on the JS you obtain.
- Using **React-Router-DOM**
```jsx
function App(){
  return (<BrowserRouter>
    <Routes>
      <Route path='/dashboard' element={<Dashboard/>}></Route>
      <Route path='/' element={<Landing/>}></Route>
    </Routes>
  </BrowserRouter>);
}

```
- How do you go to a differnt page by clicking a button without hard reload? => **useNavigate()**
#### useNavigate()
- in react router dom
- used to navigate to pages.
- Can only be used through a component inside a router eg- BrowserRouter
```jsx
function App() {
  return (
    <div>
      <BrowserRouter>
      <TopBar/>
        <Routes>
          <Route path='/dashboard' element={<Dashboard/>}></Route>
          <Route path='/' element={<Landing/>}></Route>
        </Routes>
      </BrowserRouter>
    </div>
    
  )
}

function TopBar(){
  const nav=useNavigate();
  return <div>
	  <button onClick={()=>{nav('/dashboard')}}>Dashboard</button>
	  <button onClick={()=>{nav('/')}
	  }>Landing</button>
</div>
} 

export default App

```
#### Lazy Loading
- As most users will visit only a few pages, it makes sense to load only small chuncks of site at once.
```jsx
const Landing = React.lazy(()=>{import('./components/Landing')})
const Dashboard = React.lazy(()=>{import('./components/Dashboard')})

function App() {
  return (
    <div>
      <BrowserRouter>
      <TopBar/>
        <Routes>
          <Route path='/dashboard' element={<Suspense fallback={'Loading....'}><Dashboard/></Suspense>}></Route>
          <Route path='/' element={<Suspense fallback={'Loading....'}><Landing/></Suspense>}></Route>
        </Routes>
      </BrowserRouter>
    </div>
  )
}

function TopBar(){
  const nav=useNavigate();
  return <div>
	  <button onClick={()=>{nav('/dashboard')}}>Dashboard</button>
	  <button onClick={()=>{nav('/')}
	  }>Landing</button>
  </div>
} 

export default App

```
- React.lazy loads the page only when the user goes to the page.
- The Suspense API allows to provide a default html when the component is being loaded.
## Prop Drilling, Context API
- when a prop defined here is to be used for a lower component, it needs to be passed through all the components in the path which dont need it. This is called prop drilling.
- This makes the code verbose and unappealing.
```jsx
function App() {
  const [count, setCount] = useState(0);
  return <div><Count count={count} setCount={setCount} /></div>;
}
// Count doesnot require setCount. But as Buttons requires it, it needs to pass it through.
function Count({ count, setCount }) {
  return <div>{count}
  <Buttons count={count} setCount={setCount} /></div>
}

function Buttons({ count, setCount }) {
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  );
}

```

#### Context API
- It helps in making the code more appealing by just 'teleporting' the props.
```jsx
// Context.jsx
import { createContext } from "react";
export const CountContext = createContext(0);

// => App.jsx
import './App.css'
import { CountContext } from './context';

function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <CountContext.Provider value={setCount}>
        <Count count={count}/>
      </CountContext.Provider>
    </div>
  );
}

function Count({ count }) {
  return (
    <div>
      {count}
      <Buttons count={count} />
    </div>
  );
}

function Buttons({ count }) {
  const setCount = useContext(CountContext);
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Icrement</button>
      <button onClick={() => setCount(count - 1)}>decrement</button>
    </div>
  );
}

export default App

```

## Suspense and Error Boundary
#### Suspense
- Used to handle loading screens when there is delay in rendering of component due to async calls

#### Error Boundary
- Used to handle error screens when a component throws an error.
## [Deploying Frontend](AWS - frontend deployment)

