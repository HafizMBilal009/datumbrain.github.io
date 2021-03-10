---
layout: post
title: React - Best Practices
author: Hafiz Muhammad Bilal
authorUrl: https://github.com/HafizMBilal009
date: "2021-03-10 19:50:32"
---

While working on a React App, try to follow these coding practices to have a better development experience:

## Use Visual Studio Code as Your IDE

Visual Studio Code has several features that a React developer loves. VS Code gives a lot of useful extensions to make the development environment better. For React, here some useful extensions which will assist you during development:

- Prettier
- ES Lint
- JavaScript (ES6) code snippets
- Reactjs code snippets
- Auto Import

## Use ES6 Syntax:

Clean code is always appreciated. In JavaScript, you can adopt ES6 syntax to make your code cleaner

### Write arrow functions:

```jsx
//ES5
function getSum(a, b) {
  return a + b;
}

//ES6
const getSum = (a, b) => a + b;
```

### Use template literal:

```jsx
//ES5
var name = "Bilal";
console.log("My name is " + name);

//ES6
const name = "Bilal";
console.log(`My name is ${name}`);
```

### Use `const` and `let`:

They have block scope. A variable with `const` declaration can't be changed but with `let`, It can be.

```jsx
//ES5
var fruits = ["apple", "banana"];

//ES6
let fruits = ["apple", "banana"];
fruits.push("mango");

const workingHours = 8;
```

### Object destructuring:

```jsx
var person = {
  name: "John",
  age: 40,
};

//ES5
var name = person.name;
var age = person.age;

//ES6
const { name, age } = person;
```

### Defining objects:

```jsx
var name = "John";
var age = 40;
var designations = "Full Stack Developer";
var workingHours = 8;

//ES5
var person = {
  name: name,
  age: age,
  designation: designation,
  workingHours: workingHours,
};

//ES6
const person = { name, age, designation, workingHours };
```

You will experience many features and flexibility in ES6 syntax

## Don't Forget `key` Prop With `map` in JSX:

Always add `key` prop to every JSX element while mapping from an array. Read [this article](https://dev.to/francodalessio/understanding-the-importance-of-the-key-prop-in-react-3ag7) for better understanding

```jsx
const years = [2021, 2022];

//in return function of component
<ul>
  {years.map((year, index) => (
    <li key={index}>{year}</li>
  ))}
</ul>;
```

## Component Name Should be in PascalCase:

```jsx
const helloText = () => <div>Hello</div>; //wrong

const HelloText = () => <div>Hello</div>; //correct
```

## Variable & Function Names Should be in camelCase

```jsx
const working_hours = 10; //bad approach

const workingHours = 10; //good approach

const get_sum = (a, b) => a + b; // bad approach

const getSum = (a, b) => a + b; //good approach
```

## Id & Class Names Should be in kebab-case

```html
<!--bad approach-->
<div className="hello_word" id="hello_world">Hello World</div>

<!--good approach -->
<div className="hello-word" id="hello-world">Hello World</div>
```

## Always Check `null` & `undefined` for Objects & Arrays

Neglecting `null` and `undefined` in the case of objects & arrays can lead to errors.

So, always check for them in your code

```jsx
const person = {
  name: "Haris",
  city: "Lahore",
};
console.log("Age", person.age); //rrror
console.log("Age", person.age ? person.age : 20); //correct
console.log("Age", person.age ?? 20); //correct

const oddNumbers = undefined;
console.log(oddNumbers.length); //error
console.log(oddNumbers.length ? oddNumbers.length : "Array is undefined"); //correct
console.log(oddNumbers.length ?? "Array is undefined"); //correct
```

## Avoid Inline Styling

Inline styling makes your JSX code messy. It is good to use classes & ids for styling in a separate `.css` file

```jsx
const text = <div style={{ fontWeight: "bold" }}>Happy Learing!</div>; //bad approach

const text = <div className="learning-text">Happy Learing!</div>; //good approach
```

in `.css` file:

```css
.learning-text {
  font-weight: bold;
}
```

## Avoid DOM Manipulation

Try to use React state instead of DOM manipulation as:

Bad approach:

```html
<div id="error-msg">Please enter a valid value</div>
```

```jsx
document.getElementById("error-msg").visibility = visible;
```

Good approach:

```jsx
const [isValid, setIsValid] = useState(false);

<div hidden={isValid}>Please enter a valid value</div>;
```

Set `isValid` false or true where you have logic of validating a value

## Always Remove Every Event Listener in `useEffect`

Don't forget to write cleanup function in `useEffect` to remove event listener you added before

```jsx
const printHello = () => console.log("HELLO");
useEffect(() => {
  document.addEventListener("click", printHello);
  return () => document.removeEventListener("click", printHello);
});
```

## Use a Generic Component Instead of Repeating Code

It is the best thing to make your code cleaner. Write a generic component for similar group of elements and render them on the basis of `props`
passed to it

```jsx
const Input=(props)=>{
  const [inputValue, setInputValue]=useState('');
  return(
    <label>{props.thing}</label>
    <input type='text' value={inputValue} onChange={(e)=>setInputValue(e.target.value)} />
  )
}
```

In other component you can use `Input` component as:

```jsx
<div>
  <Input thing="First Name" />
  <Input thing="Second Name" />
</div>
```

## Don’t Throw Your Files Randomly

Keep the related files in the same folder instead of making files in a single folder.

For example, if you want to create a navbar in React then you should create a folder and place `.js` & `.css` files related to the navbar in it

## Functional Components Are Recommended

If you want to render some elements and don't need to use state then use functional components instead of class components because functional components are easy to use.

Moreover, if you have an idea of **React Hooks**, then with functional components you can easily play with the state too.

## Build a Habit of Writing Helper Functions

Sometimes you need a utility at more than one time across your React App.

To deal with this scenario efficiently, Write a helper function in a separated file named `helper-functions.js`, import wherever you want to use it and call that function in it.

## Use Ternary Operator Instead of `if/else if` Statements

Using `if/else if` statements makes your code bulky. Instead try to use ternary operator where possible to make code simpler & cleaner

```jsx
//Bad approach
if (name === "Ali") {
  return 1;
} else if (name === "Bilal") {
  return 2;
} else {
  return 3;
}

//Good approach
name === "Ali" ? 1 : name === "Bilal" ? 2 : 3;
```

## Make `index.js` File Name to Minimize Importing Complexity

If you have a file named `index.js` in a directory named `actions` and you want to import action from it in your component, your import would be like this:

```jsx
import { actionName } from "src/redux/actions";
```

`actions` directory path is explained in the above import . Here you don't need to mention `index.js` after `actions` like this:

```jsx
import { actionName } from "src/redux/actions/index";
```

## Destructuring of Props

If you want to get rid of writing an object name again and again to access its properties, then destructuring of that object is the best solution for you.
Suppose your component is receiving some values like `name`, `age` and `designation` as props:

```jsx
//Bad approach
const Details = (props) => {
  return (
    <div>
      <p>{props.name}</p>
      <p>{props.age}</p>
      <p>{props.designation}</p>
    </div>
  );
};

//Good approach
const Details = ({ name, age, designation }) => {
  return (
    <div>
      <p>{name}</p>
      <p>{age}</p>
      <p>{designation}</p>
    </div>
  );
};
```

## Don't Try to Access Modified State Variable in the Same Function

In a function, if you are assigning a value to a state variable then you won't be able to access that assigned value even after it has been assigned in that function

```jsx
const Message = () => {
  const [message, setMessage] = useState("Hello World");
  const changeMessage = (messageText) => {
    setMessage("Happy Learning");
    console.log(message); //It will print Hello World on console
  };

  return <div>{message}</div>;
};
```

## Use `===` Operator instead of `==`

While comparing two values, strictly checking both values and their data types is a good practice.

```jsx
"2" == 2 ? true : false; //true
"2" === 2 ? true : false; //false
```

Now get your hands dirty with these best coding practices in React!
