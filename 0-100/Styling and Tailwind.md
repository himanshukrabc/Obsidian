## Flex
- display = flex in any tag, justify-content => ![[Pasted image 20240714204921.png]]
- This effects all the tags/components inside the tag which has flex display.
- **Tailwind**
```jsx
//CSS
	<div style={{display:'flex',justifyContent:'space-between'}}></div>
//Tailwind
	<div className='flex justify-between'}></div>
```


## Grids
```JSX
export default function Test() {
  return (
  //Equal width columns
    <div className="grid grid-cols-3">
      <div style={{ background: "green" }}></div>
      <div style={{ background: "red" }}>
        Hi there from the second div
      </div>
      <div style={{ background: "pink" }}>
        Hi there from the third div
      </div>
    </div>
  );
}
// unequal width
export default function Test() {
  return (
    <div className="grid grid-cols-12">
      <div className="col-span-5" style={{ background: "green" }}>
        <h1>Hello from the first div</h1>
      </div>
      <div className="col-span-5" style={{ background: "red" }}>
        <h1>Hi there from the second div</h1>
      </div>
      <div className="col-span-2" style={{ background: "pink" }}>
        <h1>Hi</h1>
      </div>
    </div>
  );
}
```


## Responsiveness
- Basically you show different layouts when dealing with different devices
- The devices are categorized into the following categories based on width.
- Whenever the width crosses a threshold you switch to a different layout.

| Breakpoint prefix | Minimum width | CSS                                  |
| ----------------- | ------------- | ------------------------------------ |
| `sm`              | 640px         | `@media (min-width: 640px) { ... }`  |
| `md`              | 768px         | `@media (min-width: 768px) { ... }`  |
| `lg`              | 1024px        | `@media (min-width: 1024px) { ... }` |
| `xl`              | 1280px        | `@media (min-width: 1280px) { ... }` |
| `2xl`             | 1536px        | `@media (min-width: 1536px) { ... }` |

- Tailwind uses mobile first approach. All default settings will be applied to mobile devices.
- You define different css based on prefix.

![[Pasted image 20240721104120.png]]
