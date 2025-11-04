- When we pass props through context, rerender of the props causes all the components in the path from provider to user to rerender even if they are not using the props.\
- State Management libraries help you segregate the state and components.
- This helps in removing the rerendering process.
- **Atom** - it is the smallest unit of state that can be teleported to each component.
- The following APIs are to be used ; 
	- RecoilRoot - need to wrap all components using recoil in this component.
	- atom
	- useRecoilState - like useState. => [state,useState]
	- useRecoilValue - returns just the state.
	- useSetRecoilState - returns just the setState function.
	- selector - 
```jsx
// atom -> src/store/atoms/countAtom.jsx
import { atom } from "recoil";

export const countAtom = atom({
    key:'countAtom',//unique id
    default:0// default value
});

export const evenSelector = selector({
    key :'evenSelector',
    get : ({get})=>{
        const count=get(countAtom);
        return count%2;
    }
})

// app component
import { RecoilRoot, useRecoilState, useRecoilValue, useSetRecoilState } from 'recoil';
import { countAtom } from './store/atoms/count';

function App() {
  return (
    <div>
    <RecoilRoot>
        <Count></Count>
    </RecoilRoot>
    </div>
  );
}

function Count(){
  console.log('reremder');
  return(
    <div>
        <CountRenderer/>
        <Buttons/>
        <EvenCountRenderer/>
    </div>
  );
}

function EvenCountRenderer(){
  const count=useRecoilValue(countAtom);
  if(count%2) return <div></div>;
  else return <div>It is even</div>
}

function CountRenderer() {
  const count=useRecoilValue(countAtom);
  return (
    <div>
      <b>{count}</b>
    </div>
  );
}

function Buttons() {
  const setCount = useSetRecoilState(countAtom);
  return (
    <div>
      <button onClick={() => setCount((c)=>{return c+1})}>Increment</button>
      <button onClick={() => setCount((c)=>{return c-1})}>Decrement</button>
    </div>
  );
}

export default App
```

#### When to use useState?
- When you have to use the state in the same component
- Like in the case of input fields.

#### selector
- It is used to get some result derived from the state.
- It will only change when that state variable changes.
- Works similar to useMemo().
- selector which depends on more than one atoms -
```jsx
export const filteredTodos = selector({
    key :'Todos',
    get : ({get})=>{
        const todos=get(todoAtom);
        const filter=get(filterAtom);
        return todos.filter((todo)=>{
            return todo.title.includes(filter) || todo.description.includes(filter);
        });
    }
});

useRecoilValue(filteredTodos);
```

#### Handling Asynchronous Queries
- If we want to assign value to an atom based on a query on first render, we cannot you useEffect(). This will first render the default value and then the fetched value.
- There is a lag in the loading of the website. This is fixed using the loadable hooks
```jsx
function App() {
  return <RecoilRoot>
    <MainApp />
  </RecoilRoot>
}
// will lead to double rerender => 1st display default value then the correct value
function MainApp() {
  const [networkCount, setNetworkCount] = useRecoilState(notifications)
  const totalNotificationCount = useRecoilValue(totalNotificationSelector);

  useEffect(() => {
    // fetch
    axios.get("https://sum-server.100xdevs.com/notifications")
      .then(res => {
        setNetworkCount(res.data)
      })
  }, [])

  return (
    <>
      <button>Home</button>
      <button>My network ({networkCount.networks >= 100 ? "99+" : networkCount.networks})</button>
      <button>Jobs {networkCount.jobs}</button>
      <button>Messaging ({networkCount.messaging})</button>
      <button>Notifications ({networkCount.notifications})</button>
      <button>Me ({totalNotificationCount})</button>
    </>
  )
}
```
- We cannot use useEffect() to assign default values of an atom as default value assignment function must be synchronous functions
```jsx
// Asynchronous data queries will give error
export const notifications = atom({
  key: 'notifications',
  default: async () => {
    const res = await axios.get("https://sum-server.100xdevs.com/notifications");
    return res.data;
  }
});
```

- Selectors can be used to incorporate async data into default value.

## Atom Family
- Used to create atoms dynamically.
- Returns a function that returns a new atom.
- This can also be done using an atom containing array of todos.
- However this will lead to updation of all todos when a todo is updated.
- This helps in rerendering only the updated div.
```js
import {atomFamily} from 'recoil';
import {TODOS} from './todos'
export const todosAtomFamily = atomFamily({
	key:'todosAtomFamily',
	default: id =>{
		let foundTodo = null;
		for (let i=0;i<TODOS.length;i++){
			if(TODOS[i].id == id){
				foundTodo = TODOS[i];
				break;
			}
		}
		return foundTodo;
	}
})

function App(){
	return <RecoilRoot>
		<Todo id={1}>
		<Todo id={2}>
	</RecoilRoot>
}

function Todo({id}){
	const curTodo = useRecoilValue(todosAtomFamily(id));
	return <>
		{curTodo.title}<br/>
		{curTodo.desc}<br/>
	</>
}
```


## selector Family
- Used when an atom family requires values through asynchronous calls.
``` js
import {atomFamily} from 'recoil';
export const todosAtomFamily = atomFamily({
	key:'todosAtomFamily',
	default: selectorFamily({
		key:'todosSelectorFamily',
		get: (id)=>{
			return async function({get}){
				const result = await axios.get('https://backend/id');
				return result.data.todo;
			}
		}
	})
})

function App(){
	return <RecoilRoot>
		<Todo id={1}>
		<Todo id={2}>
	</RecoilRoot>
}

function Todo({id}){
	const curTodo = useRecoilValue(todosAtomFamily(id));
	return <>
		{curTodo.title}<br/>
		{curTodo.desc}<br/>
	</>
}
```


## useRecoilStateLoadable
- used to display a loader when a async call is being made to get the default value to generate the atom.
```jsx
import { atomFamily, selectorFamily } from "recoil";
import axios from "axios";
export const todosAtomFamily = atomFamily({
  key: 'todosAtomFamily',
  default: selectorFamily({
    key: "todoSelectorFamily",
    get: (id) => async ({get}) => {
      await new Promise(r => setTimeout(r, 5000));
      const res = await axios.get(`https://sum-server.100xdevs.com/todo?id=${id}`);
      return res.data.todo;
    },
  })
});

//Todo in App
function Todo({id}) {
   const [todo, setTodo] = useRecoilStateLoadable(todosAtomFamily(id));
   if (todo.state === "loading") {
      return <div>loading</div>
   }
   if(todo.state === "hasValue") {
   return (
    <>
      {todo.contents.title}
      {todo.contents.description}
      <br />
    </>
  )}
  if(todo.state === "hasError"){
    return (
      <>
      Error while getting data
      </>
    )
  }
}
```

## useRecoilValueLoadable
- Just returns the value and not the value and function.
- Works similar to the useRecoilStateLoadable.
```jsx
function Todo({id}) {
   const todo = useRecoilValueLoadable(todosAtomFamily(id));
   if (todo.state === "loading") {
      return <div>loading</div>
   }
   if(todo.state === "hasValue"){
    return (
      <>
        {todo.contents.title}
        {todo.contents.description}
        <br />
      </>
    )
  }
   if(todo.state === "hasError"){
    return (
      <>
      Error
      </>
    )
  }
}
```