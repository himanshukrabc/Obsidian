##### **Vite** Getting the boiler plate code
-> npm create vite@latest

- npm run dev => converts react to HTML/CSS/JS using react compiler
- Every website has 2 parts - 
	1. **State** - 
		- Represents the state of the website. 
		- Used for the dynamic things in the website. eg- counter.
		- The entire website can be represented as the state.
	2. **Components** - 
		- It is a reusable dynamic HTML snippet which changes with the state.
		- It takes state as input and returns HTML.
	3. **Rendering of Components** -
		- Any change in state leads to re-rendering of the components
		- If the parent rerenders, all children rerender.

###### How react handles the returned HTML
![[Pasted image 20240624194533.png]]

