- JS manipulates HTML using the DOM making the webpage dynamic.
- DOM is a tree like representation of the webpage.
- It is independent of the language. Any lang can manipulate it.![[Pasted image 20240622162502.png]]
![[Pasted image 20240622162535.png]]
## - Document Object
- It is the topmost object of the DOM.
- It can be accessed using document keyword.
```HTML


```

## Finding Elements
#### Using Id 
-> document.getElementById("id")
#### Using TagName 
-> document.getElementByTagName("id")
#### Using ClassName 
-> document.getElementByClassName("id")
#### Using CSS Selectors 
-> document.querySelectorAll("id") => **Returns a list of nodes.**
	- [Various CSS Selectors](https://www.w3schools.com/css/css_selectors.asp)
```HTML
<!DOCTYPE html>
<html>
<body>
<h2>JavaScript HTML DOM</h2>
<p>Finding HTML Elements by Query Selector</p>

<div>Div one </div>
<p class='intro'> Paragraph one </p>
<div>Div two </div>
<p class='subheading'> Paragraph two </p>
<div> Yet another div </div>
<div id="myDIV" style="border:1px solid black;padding:8px;">
  <h2 class="example">A Heading</h2>
  <p class="example">A paragraph.</p>
  <p class="example">A paragraph.</p>
</div>
<p id="demo"></p>

<script>
  var paragraphs = document.querySelectorAll( 'p.intro' );
  paragraphs.forEach(paragraph => paragraph.style.backgroundColor = "red")

  // can pass multiple selectors as well
  const element = document.getElementById("myDIV");
  const list = element.querySelectorAll(".example");
  document.getElementById("demo").innerHTML = list.length;
</script>
</body>
</html>
```

#### Using HTML Collections 
```HTML
<!DOCTYPE html>
<html>
<head>
    <title>HTML Elements Tutorial</title>
</head>
<body>
    <h2>Finding HTML Elements Using document.anchors</h2>
    <a name="html">HTML Tutorial</a><br>
    <a name="css">CSS Tutorial</a><br>
    <a name="xml">XML Tutorial</a><br>
    <p id="demo"></p>
    <script>
        document.getElementById("demo").innerHTML = "Number of anchors are: " + document.anchors.length;
    </script>
</body>
</html>

```
- document.anchors
- document.body
- document.documentElement
- document.embeds
- document.forms
- document.head
- Document.images
- Document.links
- Document.scripts
![[Pasted image 20240622165134.png]]



## Changing the Properties
##### Changing the Values
element.**innerHTML** = "afiuhu"
##### Changing the Attributes
element._attr_name_ = _attr_value_
element.**setAttribute**(_attr_name_,_attr_value_)
=> Usually we use the 1st method as many attributes take objects as values and changing any one property is easier by chaining the keys.

## Adding and Deleting Elements
![[Pasted image 20240622165759.png]]
## Child, Sibling
![[Pasted image 20240622170007.png]]
![[Pasted image 20240622170103.png]]

## Types of Nodes
![[Pasted image 20240622170301.png]]

## Events, EventListeners
```HTML
<!DOCTYPE html>
<html>
<body>

<h2>JavaScript addEventListener()</h2>

<button id="myBtn">Try it</button>

<p>Clicking on the above button causes two alert boxes to show up.</p>

<script>
var x = document.getElementById("myBtn");
x.addEventListener('click', myFunction);
x.addEventListener('click', someOtherFunction, true);

function myFunction() {
  alert('Hello World!');
}

function someOtherFunction() {
  alert('This function was also executed!');
}
</script>

</body>
</html>

```

##### Event Propagation
- It is the way of defining the element order in which the event occurs
- If there are 2 nested elements each having the onClick event, which event will be triggered first if inner element is clicked?
- **Bubbling** - Innermost first  
- **Capturing** - Outermost first  
- By default bubbling is used.
x.addEventListener(_Event_,_eventHandlerFunction_,_useCapture_)