# Table-of-Contents

<!-- toc -->

  * [1-jsx-and-rendering](#1-jsx-and-rendering)
    + [1.1-what-is-jsx-how-is-it-different-from-html](#11-what-is-jsx-how-is-it-different-from-html)
    + [1.2-how-is-jsx-transpiled-to-javascript-can-you-demonstrate-this-with-code](#12-how-is-jsx-transpiled-to-javascript-can-you-demonstrate-this-with-code)
    + [1.3-explain-how-react-renders-elements-to-the-dom-how-does-the-virtual-dom-fit-into-this](#13-explain-how-react-renders-elements-to-the-dom-how-does-the-virtual-dom-fit-into-this)
    + [1.4-how-do-you-handle-conditions-in-jsx](#14-how-do-you-handle-conditions-in-jsx)
- [2-components](#2-components)
    + [2.1-what-is-the-difference-between-functional-and-class-components-in-react](#21-what-is-the-difference-between-functional-and-class-components-in-react)
    + [2.2-explain-the-component-lifecycle-what-are-the-methods-in-a-class-component-lifecycle](#22-explain-the-component-lifecycle-what-are-the-methods-in-a-class-component-lifecycle)
    + [2.3-how-do-you-manage-state-and-props-in-a-component](#23-how-do-you-manage-state-and-props-in-a-component)
    + [2.4-how-do-you-pass-data-from-a-parent-to-a-child-component-using-props](#24-how-do-you-pass-data-from-a-parent-to-a-child-component-using-props)
- [3-state-and-props](#3-state-and-props)
    + [3.1-what-is-the-difference-between-state-and-props-in-react-can-you-give-an-example](#31-what-is-the-difference-between-state-and-props-in-react-can-you-give-an-example)
    + [3.2-how-do-you-lift-state-from-a-child-component-to-a-parent-component](#32-how-do-you-lift-state-from-a-child-component-to-a-parent-component)
    + [3.3-how-do-you-pass-functions-as-props-to-child-components-can-you-provide-an-example](#33-how-do-you-pass-functions-as-props-to-child-components-can-you-provide-an-example)
- [4-event-handling](#4-event-handling)
    + [4.1-how-does-event-handling-work-in-react-what-are-the-differences-from-traditional-dom-events](#41-how-does-event-handling-work-in-react-what-are-the-differences-from-traditional-dom-events)
    + [4.2-how-do-you-bind-event-handlers-in-react-can-you-show-how-to-handle-a-click-event-in-a-class-component-vs-a-functional-component](#42-how-do-you-bind-event-handlers-in-react-can-you-show-how-to-handle-a-click-event-in-a-class-component-vs-a-functional-component)
    + [4.3-how-do-you-handle-controlled-and-uncontrolled-components-in-forms](#43-how-do-you-handle-controlled-and-uncontrolled-components-in-forms)
- [5-conditional-rendering](#5-conditional-rendering)
    + [5.1-how-do-you-implement-conditional-rendering-in-react](#51-how-do-you-implement-conditional-rendering-in-react)
    + [5.2-how-do-you-use-ternary-operators-and-logical-operators-for-conditional-rendering](#52-how-do-you-use-ternary-operators-and-logical-operators-for-conditional-rendering)
    + [5.3-can-you-demonstrate-conditional-rendering-using-the-operator-in-jsx](#53-can-you-demonstrate-conditional-rendering-using-the-operator-in-jsx)
- [6-lists-and-keys](#6-lists-and-keys)
    + [6.1-how-do-you-render-a-list-of-items-in-react-using-map](#61-how-do-you-render-a-list-of-items-in-react-using-map)
    + [6.2-why-is-the-key-prop-important-when-rendering-lists-in-react-what-can-go-wrong-without-it](#62-why-is-the-key-prop-important-when-rendering-lists-in-react-what-can-go-wrong-without-it)
    + [6.3-how-would-you-handle-dynamic-lists-of-components-and-how-can-you-optimize-the-re-rendering-process](#63-how-would-you-handle-dynamic-lists-of-components-and-how-can-you-optimize-the-re-rendering-process)
- [7-forms-and-user-input](#7-forms-and-user-input)
    + [7.1-how-do-you-handle-forms-in-react-what's-the-difference-between-controlled-and-uncontrolled-components](#71-how-do-you-handle-forms-in-react-whats-the-difference-between-controlled-and-uncontrolled-components)
    + [7.2-how-do-you-manage-multiple-inputs-in-a-form-using-state](#72-how-do-you-manage-multiple-inputs-in-a-form-using-state)
    + [7.3-how-would-you-implement-form-validation-in-react](#73-how-would-you-implement-form-validation-in-react)
    + [7.4-can-you-demonstrate-how-to-handle-form-submission-in-a-react-app](#74-can-you-demonstrate-how-to-handle-form-submission-in-a-react-app)
- [8-hooks](#8-hooks)
    + [8.1-what-is-the-usestate-hook-and-how-do-you-use-it-in-functional-components](#81-what-is-the-usestate-hook-and-how-do-you-use-it-in-functional-components)
    + [8.2-explain-the-useeffect-hook-how-does-it-mimic-lifecycle-methods-in-class-components](#82-explain-the-useeffect-hook-how-does-it-mimic-lifecycle-methods-in-class-components)
    + [8.3-how-do-you-use-the-usecontext-hook-to-share-state-globally](#83-how-do-you-use-the-usecontext-hook-to-share-state-globally)
    + [8.4-how-do-you-create-custom-hooks-in-react-can-you-show-an-example](#84-how-do-you-create-custom-hooks-in-react-can-you-show-an-example)
    + [8.5-how-do-you-clean-up-side-effects-using-the-useeffect-hook](#85-how-do-you-clean-up-side-effects-using-the-useeffect-hook)
- [9-context-api](#9-context-api)
    + [9.1-what-is-the-context-api-in-react-why-would-you-use-it-over-prop-drilling](#91-what-is-the-context-api-in-react-why-would-you-use-it-over-prop-drilling)
    + [9.2-how-do-you-create-and-use-a-context-in-react](#92-how-do-you-create-and-use-a-context-in-react)
    + [9.3-how-do-you-consume-a-context-in-a-functional-component](#93-how-do-you-consume-a-context-in-a-functional-component)
- [10-react-router](#10-react-router)
    + [10.1-what-is-react-router-and-how-do-you-set-up-basic-routing-in-a-react-app](#101-what-is-react-router-and-how-do-you-set-up-basic-routing-in-a-react-app)
    + [10.2-how-do-you-use-dynamic-routes-with-react-router](#102-how-do-you-use-dynamic-routes-with-react-router)
    + [10.3-how-do-you-implement-nested-routes-in-react-router](#103-how-do-you-implement-nested-routes-in-react-router)
    + [10.4-how-do-you-handle-redirects-in-react-router](#104-how-do-you-handle-redirects-in-react-router)
- [11-state-management-redux-mobx-RTK](#11-state-management-redux-mobx-rtk)
    + [11.1-what-is-redux-can-you-explain-its-core-principles-actions-reducers-store](#111-what-is-redux-can-you-explain-its-core-principles-actions-reducers-store)
    + [11.2-how-do-you-connect-redux-to-a-react-component-can-you-show-an-example](#112-how-do-you-connect-redux-to-a-react-component-can-you-show-an-example)
    + [11.3-what-are-the-differences-between-local-state-management-and-using-redux-for-state-management](#113-what-are-the-differences-between-local-state-management-and-using-redux-for-state-management)
    + [11.4-how-do-you-use-redux-thunk-or-redux-saga-for-handling-asynchronous-actions](#114-how-do-you-use-redux-thunk-or-redux-saga-for-handling-asynchronous-actions)
    + [11.5-what-is-redux-toolkit-what-are-its-advantages-over-regular-redux](#115-what-is-redux-toolkit-what-are-its-advantages-over-regular-redux)
    + [11.6-what-is-createasyncthunk-and-how-do-you-use-it-for-handling-asynchronous-actions](#116-what-is-createasyncthunk-and-how-do-you-use-it-for-handling-asynchronous-actions)
    + [11.7-what-are-the-benefits-of-using-redux-toolkit-over-vanilla-redux](#117-what-are-the-benefits-of-using-redux-toolkit-over-vanilla-redux)
- [12-error-boundaries](#12-error-boundaries)
    + [12.1-what-are-error-boundaries-in-react-why-are-they-important](#121-what-are-error-boundaries-in-react-why-are-they-important)
    + [12.2-how-do-you-implement-an-error-boundary-in-react](#122-how-do-you-implement-an-error-boundary-in-react)
    + [12.3-what-is-the-difference-between-try-catch-and-error-boundaries-in-react](#123-what-is-the-difference-between-try-catch-and-error-boundaries-in-react)
- [13-performance-optimization](#13-performance-optimization)
    + [13.1-how-do-you-prevent-unnecessary-re-renders-in-react](#131-how-do-you-prevent-unnecessary-re-renders-in-react)
    + [13.2-what-is-reactmemo-and-when-would-you-use-it](#132-what-is-reactmemo-and-when-would-you-use-it)
    + [13.3-how-do-you-implement-code-splitting-and-lazy-loading-in-a-react-app](#133-how-do-you-implement-code-splitting-and-lazy-loading-in-a-react-app)
    + [13.4-what-are-the-best-practices-for-optimizing-react-performance](#134-what-are-the-best-practices-for-optimizing-react-performance)
- [14-testing](#14-testing)
- [15-react-devtools](#15-react-devtools)
    + [15.1-how-do-you-use-react-devtools-to-debug-react-applications](#151-how-do-you-use-react-devtools-to-debug-react-applications)
    + [15.2-what-are-some-key-features-of-react-devtools-for-performance-optimization](#152-what-are-some-key-features-of-react-devtools-for-performance-optimization)
    + [15.3-how-do-you-inspect-component-state-and-props-using-react-devtools](#153-how-do-you-inspect-component-state-and-props-using-react-devtools)
- [16-advanced-patterns](#16-advanced-patterns)
    + [16.1-what-is-a-higher-order-component-hoc-and-how-do-you-use-it-in-react](#161-what-is-a-higher-order-component-hoc-and-how-do-you-use-it-in-react)
    + [16.2-can-you-explain-render-props-and-give-an-example-of-how-to-use-them](#162-can-you-explain-render-props-and-give-an-example-of-how-to-use-them)
    + [16.3-what-are-compound-components-in-react-how-do-they-work](#163-what-are-compound-components-in-react-how-do-they-work)
- [17-server-side-rendering-ssr](#17-server-side-rendering-ssr)
    + [17.1-what-is-server-side-rendering-ssr-and-how-does-it-benefit-react-applications](#171-what-is-server-side-rendering-ssr-and-how-does-it-benefit-react-applications)
    + [17.2-how-do-you-implement-ssr-with-react-what-tools-or-frameworks-would-you-use](#172-how-do-you-implement-ssr-with-react-what-tools-or-frameworks-would-you-use)
    + [17.3-what-are-the-challenges-of-ssr-with-react-and-how-do-you-overcome-them](#173-what-are-the-challenges-of-ssr-with-react-and-how-do-you-overcome-them)
- [18-typescript-with-react](#18-typescript-with-react)
    + [18.1-how-do-you-type-react-components-with-typescript](#181-how-do-you-type-react-components-with-typescript)
    + [18.2-how-do-you-type-hooks-like-usestate-and-useeffect-in-typescript](#182-how-do-you-type-hooks-like-usestate-and-useeffect-in-typescript)
    + [18.3-how-do-you-type-props-and-events-in-react-with-typescript](#183-how-do-you-type-props-and-events-in-react-with-typescript)
    + [18.4-what-are-the-benefits-of-using-typescript-with-react-and-how-does-it-improve-development](#184-what-are-the-benefits-of-using-typescript-with-react-and-how-does-it-improve-development)

<!-- tocstop -->

---

Here is **Topic 1: JSX and Rendering** with the updated format and rules:

---

## 1-jsx-and-rendering

### 1.1-what-is-jsx-how-is-it-different-from-html

**Conceptual Answer:**  
JSX (JavaScript XML) is a syntax extension for JavaScript that allows us to write HTML-like structures within JavaScript. It is not valid JavaScript but is transpiled to `React.createElement()` calls. JSX is a core feature in React because it enables developers to create components in a declarative manner that is similar to HTML, but with the power of JavaScript.

**Key Differences from HTML:**

1. **Attributes**: In JSX, you must use `className` instead of `class` because `class` is a reserved word in JavaScript.
    
2. **Event Handling**: In JSX, event names are written in camelCase. For example, `onclick` becomes `onClick`.
    
3. **Self-closing tags**: JSX requires self-closing tags for elements like `<img />` or `<input />`, while HTML allows them to be written as `<img>` or `<input>`.
    
4. **Expressions**: You can embed JavaScript expressions inside JSX by wrapping them in curly braces `{}`.
    

**Machine Coding Task:**  
Write a simple React component that renders a `div` containing a `button`. The button should say "Click me", and when clicked, it should show an alert that says "Hello, JSX!"

```jsx
function MyComponent() {
  const handleClick = () => {
    alert('Hello, JSX!');
  };

  return (
    <div>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}

export default MyComponent;
```

---

### 1.2-how-is-jsx-transpiled-to-javascript-can-you-demonstrate-this-with-code

**Conceptual Answer:**  
JSX is not natively understood by browsers. It needs to be transpiled into regular JavaScript. Tools like Babel perform this transformation. The JSX code:

```jsx
const element = <h1>Hello, world!</h1>;
```

is transpiled into:

```javascript
const element = React.createElement('h1', null, 'Hello, world!');
```

This `React.createElement()` function creates a virtual DOM element that React uses to efficiently update the real DOM when necessary.

**Machine Coding Task:**  
Write JSX code to display a `div` with "JSX to JavaScript" text. In your browser console, log the JavaScript equivalent of the JSX code (simulate using `React.createElement`).

```jsx
const element = <div>JSX to JavaScript</div>;
console.log(React.createElement('div', null, 'JSX to JavaScript'));
```

---

### 1.3-explain-how-react-renders-elements-to-the-dom-how-does-the-virtual-dom-fit-into-this

**Conceptual Answer:**  
React uses a concept called the **Virtual DOM** to optimize rendering. The Virtual DOM is a lightweight representation of the real DOM. When you create elements in React, they are initially rendered to the Virtual DOM. React compares the current Virtual DOM with the previous version (using an algorithm called "Reconciliation") and only updates the parts of the actual DOM that have changed, which makes updates more efficient.

**How React Renders to the DOM:**

1. React components are written in JSX, which is transformed into `React.createElement()` calls.
    
2. These `React.createElement()` calls generate a Virtual DOM tree.
    
3. The Virtual DOM is then compared to the real DOM to determine the minimal set of updates needed.
    
4. React updates only the necessary parts of the DOM, making the UI updates efficient.
    

**Machine Coding Task:**  
Write a simple React component that updates the background color of a `div` when a button is clicked. Use `useState` to trigger a re-render.

```jsx
import { useState } from 'react';

function BackgroundChanger() {
  const [color, setColor] = useState('white');

  const changeColor = () => {
    setColor(color === 'white' ? 'blue' : 'white');
  };

  return (
    <div style={{ backgroundColor: color, height: '100px', width: '100px' }}>
      <button onClick={changeColor}>Change Color</button>
    </div>
  );
}

export default BackgroundChanger;
```

---

### 1.4-how-do-you-handle-conditions-in-jsx

**Conceptual Answer:**  
In JSX, conditions can be handled using JavaScript expressions. You can use conditional operators such as the ternary operator or logical `&&` operator.

1. **Ternary Operator**: It allows you to return one value if the condition is true and another if false.
    
    ```jsx
    const element = isLoggedIn ? <p>Welcome back!</p> : <p>Please log in.</p>;
    ```
    
2. **Logical AND (`&&`) Operator**: It renders the second part of the expression only if the first part is true.
    
    ```jsx
    {isLoggedIn && <p>Welcome back!</p>}
    ```
    

**Machine Coding Task:**  
Write a React component that displays a message "You are logged in" if the user is logged in, and "Please log in" if not. Use a variable `isLoggedIn` to simulate this.

```jsx
function ConditionalRendering() {
  const isLoggedIn = true; // Change this value to false to test

  return (
    <div>
      {isLoggedIn ? <p>You are logged in</p> : <p>Please log in</p>}
    </div>
  );
}

export default ConditionalRendering;
```

# 2-components

### 2.1-what-is-the-difference-between-functional-and-class-components-in-react

**Conceptual Answer:**  
In React, there are two types of components: **functional components** and **class components**.

1. **Functional Components**: These are JavaScript functions that take props as an argument and return JSX. They are simpler and easier to test, and since React 16.8, they can use hooks for state management and side effects.
    
    Example:
    
    ```jsx
    function MyComponent(props) {
      return <div>{props.message}</div>;
    }
    ```
    
2. **Class Components**: These are ES6 classes that extend `React.Component` and require a `render()` method that returns JSX. They also have lifecycle methods like `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` for handling side effects.
    
    Example:
    
    ```jsx
    class MyComponent extends React.Component {
      render() {
        return <div>{this.props.message}</div>;
      }
    }
    ```
    

Functional components are now the preferred choice due to their simplicity and the power of hooks.

**Machine Coding Task:**  
Write a functional and a class component that display the same message: "Hello, React!". Compare their structure.

```jsx
// Functional Component
function FunctionalComponent() {
  return <div>Hello, React!</div>;
}

// Class Component
class ClassComponent extends React.Component {
  render() {
    return <div>Hello, React!</div>;
  }
}
```

---

### 2.2-explain-the-component-lifecycle-what-are-the-methods-in-a-class-component-lifecycle

**Conceptual Answer:**  
A **component lifecycle** refers to the series of methods React calls at different stages of a component's existence. These stages include mounting, updating, and unmounting.

In class components, the lifecycle methods are:

1. **Mounting** (when the component is created and inserted into the DOM):
    
    - `constructor`: Used to initialize the state and bind methods.
        
    - `componentDidMount`: Called once after the component has been mounted. You can make API calls here.
        
2. **Updating** (when the component's state or props change):
    
    - `shouldComponentUpdate`: Determines whether the component should update based on changes to state or props.
        
    - `componentDidUpdate`: Called after the component has updated. You can perform operations like DOM manipulation here.
        
3. **Unmounting** (when the component is removed from the DOM):
    
    - `componentWillUnmount`: Called right before the component is removed from the DOM. It’s used for cleanup, such as invalidating timers or canceling network requests.
        

**Machine Coding Task:**  
Write a class component that logs messages during each lifecycle method (mounting, updating, and unmounting).

```jsx
class LifecycleDemo extends React.Component {
  constructor(props) {
    super(props);
    console.log("Constructor: Initializing component");
  }

  componentDidMount() {
    console.log("componentDidMount: Component has mounted");
  }

  shouldComponentUpdate(nextProps, nextState) {
    console.log("shouldComponentUpdate: Should the component update?");
    return true; // You can return false to prevent re-render
  }

  componentDidUpdate(prevProps, prevState) {
    console.log("componentDidUpdate: Component has updated");
  }

  componentWillUnmount() {
    console.log("componentWillUnmount: Component is about to be unmounted");
  }

  render() {
    return <div>Lifecycle Demo</div>;
  }
}

export default LifecycleDemo;
```

---

### 2.3-how-do-you-manage-state-and-props-in-a-component

**Conceptual Answer:**  
In React, **state** and **props** are used to manage data in components.

1. **State**: It is data that belongs to a component. State is mutable and can be changed. Each component can have its own state, which is typically used for managing user interactions or component-specific data.
    
    Example of managing state in a functional component using `useState` hook:
    
    ```jsx
    import { useState } from 'react';
    
    function Counter() {
      const [count, setCount] = useState(0);
    
      const increment = () => setCount(count + 1);
    
      return (
        <div>
          <p>{count}</p>
          <button onClick={increment}>Increment</button>
        </div>
      );
    }
    ```
    
2. **Props**: They are passed from a parent component to a child component and are read-only. Props allow data to flow down the component tree.
    
    Example of passing props:
    
    ```jsx
    function ParentComponent() {
      return <ChildComponent message="Hello from parent!" />;
    }
    
    function ChildComponent(props) {
      return <div>{props.message}</div>;
    }
    ```
    

**Machine Coding Task:**  
Write a functional component that uses `useState` to manage a counter. Pass the counter value as a prop to a child component.

```jsx
function ParentComponent() {
  const [counter, setCounter] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCounter(counter + 1)}>Increment</button>
      <ChildComponent counter={counter} />
    </div>
  );
}

function ChildComponent({ counter }) {
  return <div>Current counter: {counter}</div>;
}
```

---

### 2.4-how-do-you-pass-data-from-a-parent-to-a-child-component-using-props

**Conceptual Answer:**  
In React, **props** (short for properties) are used to pass data from a parent component to a child component. Props are read-only and cannot be modified by the child component.

The parent component can pass any kind of data (strings, numbers, arrays, functions, etc.) to the child component as props.

Example:

```jsx
function ParentComponent() {
  const parentData = "Hello from Parent";

  return <ChildComponent data={parentData} />;
}

function ChildComponent(props) {
  return <div>{props.data}</div>;
}
```

In this example, the `ParentComponent` passes the string `parentData` as a prop named `data` to the `ChildComponent`.

**Machine Coding Task:**  
Write a functional component where the parent passes a message to the child, and the child displays the message.

```jsx
function ParentComponent() {
  const message = "This is a message from the parent";
  return <ChildComponent message={message} />;
}

function ChildComponent({ message }) {
  return <p>{message}</p>;
}
```


# 3-state-and-props

### 3.1-what-is-the-difference-between-state-and-props-in-react-can-you-give-an-example

**Conceptual Answer:**  
In React, **state** and **props** are both used for managing data in components, but they serve different purposes:

1. **State**:
    
    - State is used for data that is **local** to a component and can change over time.
        
    - It is **mutable**, and a component can modify its state.
        
    - State is typically used for storing values like user input, component visibility, or UI changes.
        
2. **Props**:
    
    - Props are used to pass data from a **parent component** to a **child component**.
        
    - Props are **immutable**, meaning the child component cannot change them; they are set by the parent.
        
    - Props are used to pass configuration, data, or event handlers to child components.
        

**Example:**

```jsx
function ParentComponent() {
  const parentMessage = "Hello from Parent!";
  return <ChildComponent message={parentMessage} />;
}

function ChildComponent(props) {
  return <div>{props.message}</div>;
}
```

In this example, `message` is a prop passed from the parent to the child.

**Machine Coding Task:**  
Write a component that uses both state and props. The parent passes a prop to the child, and the child has a local state that updates when a button is clicked.

```jsx
function ParentComponent() {
  const message = "Message from parent";

  return <ChildComponent parentMessage={message} />;
}

function ChildComponent({ parentMessage }) {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{parentMessage}</p>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

---

### 3.2-how-do-you-lift-state-from-a-child-component-to-a-parent-component

**Conceptual Answer:**  
In React, **lifting state up** refers to moving the state from a child component to a parent component. This is done when the parent needs to control or respond to changes in the child’s state. The child communicates changes to the parent through a function passed as a prop, and the parent updates its state accordingly.

**Example:**

```jsx
function ParentComponent() {
  const [data, setData] = useState("Initial Data");

  const handleDataChange = (newData) => {
    setData(newData);
  };

  return (
    <div>
      <ChildComponent data={data} onDataChange={handleDataChange} />
    </div>
  );
}

function ChildComponent({ data, onDataChange }) {
  const updateData = () => {
    onDataChange("Updated Data");
  };

  return (
    <div>
      <p>{data}</p>
      <button onClick={updateData}>Update Data</button>
    </div>
  );
}
```

In this example, `onDataChange` is a function passed from the parent to the child, and the child calls it to update the parent’s state.

**Machine Coding Task:**  
Write a parent and child component where the child updates the parent’s state when a button is clicked.

```jsx
function ParentComponent() {
  const [message, setMessage] = useState("Hello!");

  const handleMessageChange = (newMessage) => {
    setMessage(newMessage);
  };

  return (
    <div>
      <ChildComponent message={message} onMessageChange={handleMessageChange} />
    </div>
  );
}

function ChildComponent({ message, onMessageChange }) {
  return (
    <div>
      <p>{message}</p>
      <button onClick={() => onMessageChange("Hello from Child!")}>Change Message</button>
    </div>
  );
}
```

---

### 3.3-how-do-you-pass-functions-as-props-to-child-components-can-you-provide-an-example

**Conceptual Answer:**  
In React, you can pass **functions** as props from a parent component to a child component. This allows the child to invoke functions in the parent, typically to update state or trigger actions in the parent.

**Example:**

```jsx
function ParentComponent() {
  const showAlert = () => {
    alert("Hello from Parent!");
  };

  return <ChildComponent onClick={showAlert} />;
}

function ChildComponent({ onClick }) {
  return <button onClick={onClick}>Click me</button>;
}
```

In this example, `onClick` is a function passed from the parent to the child. When the button is clicked, it triggers the `showAlert` function in the parent.

**Machine Coding Task:**  
Write a parent component that passes a function to a child component to increment a count. The child component will call the parent’s function when a button is clicked.

```jsx
function ParentComponent() {
  const [count, setCount] = useState(0);

  const incrementCount = () => {
    setCount(count + 1);
  };

  return <ChildComponent increment={incrementCount} count={count} />;
}

function ChildComponent({ increment, count }) {
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```


# 4-event-handling

### 4.1-how-does-event-handling-work-in-react-what-are-the-differences-from-traditional-dom-events

**Conceptual Answer:**  
In React, **event handling** works similarly to traditional DOM events, but with some differences:

1. **Event Naming**: In React, event names are written in camelCase (e.g., `onClick`, `onChange`) rather than lowercase (e.g., `onclick`, `onchange`).
    
2. **Event Binding**: In React, event handlers are typically passed as functions, rather than string values. React automatically binds the `this` context to the handler in class components, and in functional components, you can use arrow functions or `useCallback` to manage event handlers.
    
3. **Synthetic Events**: React uses a **SyntheticEvent** system, which wraps the native DOM events to provide a consistent interface across all browsers. It is normalized for cross-browser compatibility, making the events behave consistently.
    
4. **Preventing Default Behavior**: You can use `event.preventDefault()` and `event.stopPropagation()` just like in traditional JavaScript, but React also provides an optimized way of doing so.
    

**Machine Coding Task:**  
Write a functional component where a button click increases a counter and a second button resets the counter.

```jsx
function EventHandlerDemo() {
  const [count, setCount] = useState(0);

  const handleIncrement = () => setCount(count + 1);
  const handleReset = () => setCount(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
      <button onClick={handleReset}>Reset</button>
    </div>
  );
}
```

---

### 4.2-how-do-you-bind-event-handlers-in-react-can-you-show-how-to-handle-a-click-event-in-a-class-component-vs-a-functional-component

**Conceptual Answer:**  
In React, event handlers can be bound differently in **class components** and **functional components**.

1. **Class Component**:  
    In class components, you bind event handlers in the `constructor` or directly within the method using arrow functions.
    
    Example with binding in the constructor:
    
    ```jsx
    class Counter extends React.Component {
      constructor(props) {
        super(props);
        this.state = { count: 0 };
        this.handleClick = this.handleClick.bind(this); // binding here
      }
    
      handleClick() {
        this.setState({ count: this.state.count + 1 });
      }
    
      render() {
        return <button onClick={this.handleClick}>Increment</button>;
      }
    }
    ```
    
2. **Functional Component**:  
    In functional components, you don't need explicit binding since you can use arrow functions or hooks.
    
    Example with functional component:
    
    ```jsx
    function Counter() {
      const [count, setCount] = useState(0);
    
      const handleClick = () => setCount(count + 1);
    
      return <button onClick={handleClick}>Increment</button>;
    }
    ```
    

**Machine Coding Task:**  
Write a class component and a functional component that each increase a counter when a button is clicked.

```jsx
// Class Component
class CounterClass extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>{this.state.count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }
}

// Functional Component
function CounterFunctional() {
  const [count, setCount] = useState(0);
  const handleClick = () => setCount(count + 1);

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```

---

### 4.3-how-do-you-handle-controlled-and-uncontrolled-components-in-forms

**Conceptual Answer:**  
In React, you can handle **controlled** and **uncontrolled components** in forms:

1. **Controlled Components**:  
    A controlled component is one where the form data is controlled by React state. You update the state whenever the input changes, and the value of the input field is tied to the state.
    
    Example:
    
    ```jsx
    function ControlledForm() {
      const [value, setValue] = useState("");
    
      const handleChange = (e) => setValue(e.target.value);
    
      return (
        <form>
          <input type="text" value={value} onChange={handleChange} />
          <button type="submit">Submit</button>
        </form>
      );
    }
    ```
    
2. **Uncontrolled Components**:  
    An uncontrolled component is one where the form data is handled by the DOM itself, not by React state. You use a `ref` to get the value when needed.
    
    Example:
    
    ```jsx
    function UncontrolledForm() {
      const inputRef = useRef();
    
      const handleSubmit = (e) => {
        e.preventDefault();
        alert(inputRef.current.value);
      };
    
      return (
        <form onSubmit={handleSubmit}>
          <input ref={inputRef} type="text" />
          <button type="submit">Submit</button>
        </form>
      );
    }
    ```
    

**Machine Coding Task:**  
Write a controlled form component with a text input and an uncontrolled form component with a text input. In both components, display the entered text when the form is submitted.

```jsx
// Controlled Form
function ControlledForm() {
  const [value, setValue] = useState("");

  const handleChange = (e) => setValue(e.target.value);

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" value={value} onChange={handleChange} />
      <button type="submit">Submit</button>
    </form>
  );
}

// Uncontrolled Form
function UncontrolledForm() {
  const inputRef = useRef();

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(inputRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} type="text" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

This concludes **Topic 4: Event Handling**. Let me know if you'd like to continue with the next topic!

# 5-conditional-rendering

### 5.1-how-do-you-implement-conditional-rendering-in-react

**Conceptual Answer:**  
Conditional rendering in React allows you to render different UI elements based on specific conditions. You can use **JavaScript expressions** inside JSX to determine which component or element should be displayed.

You can implement conditional rendering using:

1. **Ternary Operator**: Used to render one of two elements depending on a condition.
    
2. **Logical AND (`&&`) Operator**: Used to render an element only when a condition is true.
    
3. **If/Else**: You can also use the traditional if/else approach, but it needs to be inside a function or class method since JSX expressions must be evaluated.
    

**Example with Ternary Operator:**

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <p>Welcome back!</p> : <p>Please log in</p>}
    </div>
  );
}
```

**Machine Coding Task:**  
Write a component that displays "You are logged in" if `isLoggedIn` is `true`, and "Please log in" if `isLoggedIn` is `false`.

```jsx
function ConditionalRendering() {
  const isLoggedIn = true; // Change this to false to test

  return (
    <div>
      {isLoggedIn ? <p>You are logged in</p> : <p>Please log in</p>}
    </div>
  );
}
```

---

### 5.2-how-do-you-use-ternary-operators-and-logical-operators-for-conditional-rendering

**Conceptual Answer:**  
In React, the **ternary operator** (`condition ? true : false`) and **logical AND (`&&`) operator** are often used to conditionally render elements.

1. **Ternary Operator**:
    
    - The ternary operator is useful when you want to choose between two expressions based on a condition.
        
    
    Example:
    
    ```jsx
    {isLoggedIn ? <p>Welcome back!</p> : <p>Please log in</p>}
    ```
    
2. **Logical AND (`&&`) Operator**:
    
    - The logical AND (`&&`) operator renders the second part of the expression only if the first part is `true`. This is useful for conditionally rendering a single element or when you want to render something only if a condition is met.
        
    
    Example:
    
    ```jsx
    {isLoggedIn && <p>Welcome back!</p>}
    ```
    

**Machine Coding Task:**  
Write a component that uses both the ternary operator and logical AND operator to conditionally render messages based on the value of `isLoggedIn`.

```jsx
function ConditionalRendering() {
  const isLoggedIn = true; // Change this to false to test

  return (
    <div>
      {isLoggedIn ? <p>Welcome back!</p> : <p>Please log in</p>}
      {isLoggedIn && <p>You have access to the dashboard</p>}
    </div>
  );
}
```

---

### 5.3-can-you-demonstrate-conditional-rendering-using-the-operator-in-jsx

**Conceptual Answer:**  
You can use the **logical AND (`&&`)** operator to conditionally render an element. The second expression after the `&&` will be rendered only if the condition is `true`.

**Example:**

```jsx
function Dashboard({ isAdmin }) {
  return (
    <div>
      {isAdmin && <button>Delete User</button>}
    </div>
  );
}
```

In this example, the "Delete User" button will only render if `isAdmin` is `true`.

**Machine Coding Task:**  
Write a component that conditionally renders a message if the user is an admin, using the logical AND (`&&`) operator.

```jsx
function AdminPanel() {
  const isAdmin = true; // Change to false to test

  return (
    <div>
      {isAdmin && <p>Welcome, Admin!</p>}
      {!isAdmin && <p>You do not have admin privileges.</p>}
    </div>
  );
}
```

---

# 6-lists-and-keys

### 6.1-how-do-you-render-a-list-of-items-in-react-using-map

**Conceptual Answer:**  
In React, you can render a list of items using the **`map()`** method, which iterates over an array and returns a new array of JSX elements.

**Example:**

```jsx
function ItemList() {
  const items = ['Apple', 'Banana', 'Orange'];

  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}
```

In this example, `items.map()` creates a list of `<li>` elements, each containing an item from the array.

**Machine Coding Task:**  
Write a component that renders a list of numbers, each wrapped in an `<li>` tag.

```jsx
function NumberList() {
  const numbers = [1, 2, 3, 4, 5];

  return (
    <ul>
      {numbers.map((number) => (
        <li key={number}>{number}</li>
      ))}
    </ul>
  );
}
```

---

### 6.2-why-is-the-key-prop-important-when-rendering-lists-in-react-what-can-go-wrong-without-it

**Conceptual Answer:**  
The **`key`** prop is crucial when rendering lists in React. It helps React identify which items in the list have changed, been added, or been removed. Without a `key`, React will not be able to track individual list items efficiently, leading to unnecessary re-renders or incorrect updates.

1. **Key Prop**: React uses the `key` to optimize the rendering process. The `key` should be unique among siblings to ensure correct component matching.
    
2. **Without Key**: Without a `key`, React will rely on the index of the array, which can cause problems, especially if the list items are reordered or dynamically changed.
    

**Example (Without Key):**

```jsx
function ItemList() {
  const items = ['Apple', 'Banana', 'Orange'];

  return (
    <ul>
      {items.map((item) => (
        <li>{item}</li>
      ))}
    </ul>
  );
}
```

In this case, React will give a warning and might not optimize rendering efficiently.

**Machine Coding Task:**  
Modify the previous `ItemList` component to use the `key` prop to ensure optimal rendering.

```jsx
function ItemList() {
  const items = ['Apple', 'Banana', 'Orange'];

  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}
```

---

### 6.3-how-would-you-handle-dynamic-lists-of-components-and-how-can-you-optimize-the-re-rendering-process

**Conceptual Answer:**  
Handling dynamic lists in React requires updating the list based on state or props changes. The key to optimizing re-renders is to use the **`key`** prop properly, which helps React identify which elements need to be updated when the list changes.

1. **Stateful Lists**: When a list is dynamic, the list items might change or be reordered. The `key` ensures React only re-renders changed items.
    
2. **Optimizing Re-renders**: You can optimize re-renders by using techniques such as:
    
    - Using **`React.memo`** to memoize list items that don't change.
        
    - Ensuring that list items have unique and stable `keys`.
        

**Example of Dynamic List (With Key Prop):**

```jsx
function DynamicList() {
  const [items, setItems] = useState(['Apple', 'Banana', 'Orange']);

  const addItem = () => {
    setItems([...items, 'Grapes']);
  };

  return (
    <div>
      <button onClick={addItem}>Add Item</button>
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Machine Coding Task:**  
Write a component that renders a list of names and allows the user to add a name to the list dynamically. Use the `key` prop to ensure optimal rendering.

```jsx
function DynamicList() {
  const [names, setNames] = useState(['Alice', 'Bob', 'Charlie']);

  const addName = () => {
    const newName = prompt('Enter a new name');
    setNames([...names, newName]);
  };

  return (
    <div>
      <button onClick={addName}>Add Name</button>
      <ul>
        {names.map((name, index) => (
          <li key={index}>{name}</li>
        ))}
      </ul>
    </div>
  );
}
```


# 7-forms-and-user-input

### 7.1-how-do-you-handle-forms-in-react-what's-the-difference-between-controlled-and-uncontrolled-components

**Conceptual Answer:**  
In React, forms are handled by managing the form data with **state**. React provides two approaches for handling form inputs: **controlled components** and **uncontrolled components**.

1. **Controlled Components**:
    
    - In a controlled component, the form element’s value is controlled by the state.
        
    - The form element updates the state whenever the user interacts with it.
        
    - This gives you full control over the form data and is preferred for most scenarios.
        
    
    **Example:**
    
    ```jsx
    function ControlledForm() {
      const [inputValue, setInputValue] = useState("");
    
      const handleChange = (e) => {
        setInputValue(e.target.value);
      };
    
      return (
        <form>
          <input type="text" value={inputValue} onChange={handleChange} />
          <button type="submit">Submit</button>
        </form>
      );
    }
    ```
    
2. **Uncontrolled Components**:
    
    - In an uncontrolled component, form data is handled by the DOM itself.
        
    - You use **refs** to get the value of the form element when needed.
        
    
    **Example:**
    
    ```jsx
    function UncontrolledForm() {
      const inputRef = useRef();
    
      const handleSubmit = (e) => {
        e.preventDefault();
        alert(inputRef.current.value);
      };
    
      return (
        <form onSubmit={handleSubmit}>
          <input ref={inputRef} type="text" />
          <button type="submit">Submit</button>
        </form>
      );
    }
    ```
    

**Machine Coding Task:**  
Write a form component that uses both controlled and uncontrolled components to manage an input.

```jsx
function CombinedForm() {
  const [controlledValue, setControlledValue] = useState("");
  const uncontrolledRef = useRef();

  const handleControlledChange = (e) => {
    setControlledValue(e.target.value);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(`Controlled Value: ${controlledValue}, Uncontrolled Value: ${uncontrolledRef.current.value}`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" value={controlledValue} onChange={handleControlledChange} />
      <input ref={uncontrolledRef} type="text" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### 7.2-how-do-you-manage-multiple-inputs-in-a-form-using-state

**Conceptual Answer:**  
When managing multiple inputs in a form, you can store the values of the inputs in an **object** in the state, where each input has its own key-value pair. This approach allows you to easily handle multiple inputs by updating the corresponding state property for each input.

**Example:**

```jsx
function MultiInputForm() {
  const [formData, setFormData] = useState({
    firstName: "",
    lastName: "",
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prevState) => ({
      ...prevState,
      [name]: value,
    }));
  };

  return (
    <form>
      <input
        type="text"
        name="firstName"
        value={formData.firstName}
        onChange={handleChange}
      />
      <input
        type="text"
        name="lastName"
        value={formData.lastName}
        onChange={handleChange}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Machine Coding Task:**  
Write a form component that handles multiple inputs such as `email`, `password`, and `username`, and displays the form values when the form is submitted.

```jsx
function MultiInputForm() {
  const [formData, setFormData] = useState({
    email: "",
    password: "",
    username: "",
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prevState) => ({
      ...prevState,
      [name]: value,
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(JSON.stringify(formData));
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
      />
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleChange}
      />
      <input
        type="text"
        name="username"
        value={formData.username}
        onChange={handleChange}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### 7.3-how-would-you-implement-form-validation-in-react

**Conceptual Answer:**  
Form validation in React can be implemented by checking the form input values before submitting the form. You can use state to track validation errors and conditionally render error messages.

1. **Simple Validation**: You can use simple `if` statements or regular expressions to validate inputs.
    
2. **Conditional Rendering**: Display error messages only when validation fails.
    
3. **Submit Handling**: Prevent form submission if validation fails.
    

**Example:**

```jsx
function FormWithValidation() {
  const [formData, setFormData] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prevState) => ({
      ...prevState,
      [name]: value,
    }));
  };

  const validateForm = () => {
    let formErrors = {};
    if (!formData.email) formErrors.email = "Email is required";
    if (!formData.password) formErrors.password = "Password is required";
    return formErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const formErrors = validateForm();
    if (Object.keys(formErrors).length === 0) {
      alert("Form submitted successfully");
    } else {
      setErrors(formErrors);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
      />
      {errors.email && <p>{errors.email}</p>}
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleChange}
      />
      {errors.password && <p>{errors.password}</p>}
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### 7.4-can-you-demonstrate-how-to-handle-form-submission-in-a-react-app

**Conceptual Answer:**  
Handling form submission in React involves capturing the `submit` event, preventing the default browser action (using `e.preventDefault()`), and processing the form data. This can be done by submitting the form values to an API, updating the state, or any other necessary action.

**Example:**

```jsx
function SimpleForm() {
  const [inputValue, setInputValue] = useState("");

  const handleChange = (e) => {
    setInputValue(e.target.value);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(`Form submitted with input: ${inputValue}`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={inputValue}
        onChange={handleChange}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Machine Coding Task:**  
Write a form that handles the submission and logs the form data to the console when the submit button is clicked.

```jsx
function SimpleForm() {
  const [inputValue, setInputValue] = useState("");

  const handleChange = (e) => {
    setInputValue(e.target.value);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Form submitted with value:", inputValue);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={inputValue}
        onChange={handleChange}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

# 8-hooks

### 8.1-what-is-the-usestate-hook-and-how-do-you-use-it-in-functional-components

**Conceptual Answer:**  
The **`useState`** hook is a fundamental hook in React that allows functional components to have local state. It provides a state variable and a function to update it. This is a replacement for the `this.state` and `this.setState` used in class components.

**Syntax:**

```jsx
const [state, setState] = useState(initialValue);
```

- **state**: The current state value.
    
- **setState**: A function to update the state.
    
- **initialValue**: The initial value of the state.
    

**Example:**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

**Machine Coding Task:**  
Write a component that allows users to increment and decrement a counter using `useState`.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

---

### 8.2-explain-the-useeffect-hook-how-does-it-mimic-lifecycle-methods-in-class-components

**Conceptual Answer:**  
The **`useEffect`** hook is used to perform side effects in functional components. It can mimic the behavior of lifecycle methods like `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` in class components.

- **`useEffect`** runs after the render and is useful for fetching data, subscribing to events, or manually modifying the DOM.
    
- By providing a **dependency array** (`[]`), you can control when the effect runs, just like lifecycle methods in class components.
    

**Syntax:**

```jsx
useEffect(() => {
  // Side effect logic
}, [dependencies]);
```

- If you pass an empty array `[]`, the effect runs only once after the initial render, similar to `componentDidMount`.
    
- If you pass dependencies, the effect will run when those dependencies change, similar to `componentDidUpdate`.
    
- If you return a cleanup function inside the `useEffect` callback, it mimics `componentWillUnmount`.
    

**Example:**

```jsx
import { useState, useEffect } from 'react';

function FetchData() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch('https://api.example.com/data')
      .then((response) => response.json())
      .then((data) => setData(data));
  }, []); // Empty dependency array makes this run only once, like componentDidMount

  return (
    <div>
      <ul>
        {data.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Machine Coding Task:**  
Write a component that fetches data from an API and displays it using `useEffect` and `useState`.

```jsx
import { useState, useEffect } from 'react';

function FetchData() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/posts')
      .then((response) => response.json())
      .then((data) => setData(data));
  }, []);

  return (
    <div>
      <h2>Posts</h2>
      <ul>
        {data.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

### 8.3-how-do-you-use-the-usecontext-hook-to-share-state-globally

**Conceptual Answer:**  
The **`useContext`** hook allows you to consume context values in functional components. It helps you share state or values globally without prop drilling, i.e., passing props down multiple levels of components.

To use the `useContext` hook:

1. Create a **context** using `React.createContext()`.
    
2. Use the **`Provider`** component to provide the context value to the component tree.
    
3. Use **`useContext`** to access the context value in child components.
    

**Example:**

```jsx
import { createContext, useState, useContext } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemeSwitcher() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <div>
      <p>Current Theme: {theme}</p>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}

function App() {
  return (
    <ThemeProvider>
      <ThemeSwitcher />
    </ThemeProvider>
  );
}
```

**Machine Coding Task:**  
Write a component that uses `useContext` to toggle between light and dark themes globally.

```jsx
import { createContext, useState, useContext } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemeSwitcher() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <div>
      <p>Current Theme: {theme}</p>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}

function App() {
  return (
    <ThemeProvider>
      <ThemeSwitcher />
    </ThemeProvider>
  );
}
```

---

### 8.4-how-do-you-create-custom-hooks-in-react-can-you-show-an-example

**Conceptual Answer:**  
A **custom hook** in React is a function that allows you to reuse stateful logic across components. It can use built-in hooks like `useState`, `useEffect`, and others. Custom hooks start with the prefix `use` to follow React’s naming conventions.

**Example:**

```jsx
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return { count, increment, decrement };
}

function Counter() {
  const { count, increment, decrement } = useCounter();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

**Machine Coding Task:**  
Write a custom hook `useInput` that manages the value of an input field. Use it in a form to manage the input state.

```jsx
function useInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  const handleChange = (e) => setValue(e.target.value);

  return {
    value,
    handleChange,
  };
}

function Form() {
  const { value, handleChange } = useInput('');

  return (
    <form>
      <input type="text" value={value} onChange={handleChange} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### 8.5-how-do-you-clean-up-side-effects-using-the-useeffect-hook

**Conceptual Answer:**  
You can clean up side effects in React using the **`useEffect`** hook by returning a cleanup function. This function will be executed when the component is unmounted or before the effect runs again.

**Example:**

```jsx
useEffect(() => {
  const interval = setInterval(() => {
    console.log("Timer is running...");
  }, 1000);

  return () => {
    clearInterval(interval); // Cleanup the interval when the component is unmounted
  };
}, []);
```

The cleanup function is useful for clearing intervals, cancelling network requests, or unsubscribing from events.

**Machine Coding Task:**  
Write a component that starts a timer when mounted and clears it when unmounted using `useEffect`.

```jsx
import { useState, useEffect } from 'react';

function Timer() {
  const [time, setTime] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => setTime((prevTime) => prevTime + 1), 1000);

    return () => clearInterval(interval); // Cleanup on unmount
  }, []);

  return <div>Time: {time}s</div>;
}
```

---
# 9-context-api

### 9.1-what-is-the-context-api-in-react-why-would-you-use-it-over-prop-drilling

**Conceptual Answer:**  
The **Context API** in React provides a way to share values (such as state or functions) across the component tree without having to explicitly pass them down through every level via props. It is useful when multiple components at different nesting levels need access to the same data.

- **Prop Drilling** refers to passing props through many levels of components, even if they are not directly needed by intermediate components. This can lead to cumbersome and hard-to-maintain code.
    
- **Context API** solves this problem by providing a global state that can be accessed by any component that consumes the context, without the need for prop drilling.
    

**Key Features of Context API:**

1. **`createContext`**: Used to create a Context object.
    
2. **`Provider`**: Provides the context value to the component tree.
    
3. **`Consumer` or `useContext`**: Allows components to access the context value.
    

**Example:**

```jsx
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemeSwitcher() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <div>
      <p>Current Theme: {theme}</p>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}

function App() {
  return (
    <ThemeProvider>
      <ThemeSwitcher />
    </ThemeProvider>
  );
}
```

**Machine Coding Task:**  
Write a component that uses the Context API to manage a language setting (`English` or `Spanish`) globally and allows users to switch between them.

```jsx
const LanguageContext = createContext();

function LanguageProvider({ children }) {
  const [language, setLanguage] = useState('English');

  return (
    <LanguageContext.Provider value={{ language, setLanguage }}>
      {children}
    </LanguageContext.Provider>
  );
}

function LanguageSwitcher() {
  const { language, setLanguage } = useContext(LanguageContext);

  return (
    <div>
      <p>Current Language: {language}</p>
      <button onClick={() => setLanguage(language === 'English' ? 'Spanish' : 'English')}>
        Switch Language
      </button>
    </div>
  );
}

function App() {
  return (
    <LanguageProvider>
      <LanguageSwitcher />
    </LanguageProvider>
  );
}
```

---

### 9.2-how-do-you-create-and-use-a-context-in-react

**Conceptual Answer:**  
To create and use a context in React, follow these steps:

1. **Create a Context**: Use `createContext()` to create a context object.
    
2. **Provide the Context**: Use the `Provider` component to make the context available to all components in the component tree.
    
3. **Consume the Context**: Use `useContext` in functional components to consume the context value.
    

**Example:**

```jsx
const UserContext = createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

function UserProfile() {
  const { user } = useContext(UserContext);

  return <div>{user ? `Welcome, ${user.name}` : 'No user logged in'}</div>;
}

function App() {
  return (
    <UserProvider>
      <UserProfile />
    </UserProvider>
  );
}
```

**Machine Coding Task:**  
Write a component that uses the Context API to manage user authentication (`loggedIn` or `loggedOut`) and displays the user status accordingly.

```jsx
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [loggedIn, setLoggedIn] = useState(false);

  return (
    <AuthContext.Provider value={{ loggedIn, setLoggedIn }}>
      {children}
    </AuthContext.Provider>
  );
}

function AuthStatus() {
  const { loggedIn, setLoggedIn } = useContext(AuthContext);

  return (
    <div>
      <p>Status: {loggedIn ? 'Logged In' : 'Logged Out'}</p>
      <button onClick={() => setLoggedIn(!loggedIn)}>Toggle Login</button>
    </div>
  );
}

function App() {
  return (
    <AuthProvider>
      <AuthStatus />
    </AuthProvider>
  );
}
```

---

### 9.3-how-do-you-consume-a-context-in-a-functional-component

**Conceptual Answer:**  
To consume a context in a functional component, use the **`useContext`** hook, which gives access to the context value. You can pass the context object created by `createContext` into `useContext` to retrieve the value.

**Example:**

```jsx
const ThemeContext = createContext('light');

function ThemeComponent() {
  const theme = useContext(ThemeContext);

  return <div>Current Theme: {theme}</div>;
}

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ThemeComponent />
    </ThemeContext.Provider>
  );
}
```

In this example, `ThemeComponent` consumes the `ThemeContext` and displays the current theme.

**Machine Coding Task:**  
Write a component that consumes a theme context to display a background color based on the theme (either `light` or `dark`).

```jsx
const ThemeContext = createContext('light');

function ThemeDisplay() {
  const theme = useContext(ThemeContext);

  const themeStyles = theme === 'dark' ? { backgroundColor: 'black', color: 'white' } : {};

  return <div style={themeStyles}>This is the {theme} theme.</div>;
}

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ThemeDisplay />
    </ThemeContext.Provider>
  );
}
```

---
# 10-react-router

### 10.1-what-is-react-router-and-how-do-you-set-up-basic-routing-in-a-react-app

**Conceptual Answer:**  
**React Router** is a library used to handle routing in React applications. It allows you to define multiple routes in your app and navigate between them without reloading the page, providing a smooth single-page application (SPA) experience.

To set up basic routing in a React app, follow these steps:

1. **Install React Router**:
    
    ```bash
    npm install react-router-dom
    ```
    
2. **Set up Router**:
    
    - Use the **`BrowserRouter`** to wrap the app, which enables the routing functionality.
        
    - Use **`Route`** components to define the paths and the components to render for each path.
        
    - Use **`Link`** or **`NavLink`** to create navigable links.
        

**Example:**

```jsx
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

function Home() {
  return <h2>Home Page</h2>;
}

function About() {
  return <h2>About Page</h2>;
}

function App() {
  return (
    <Router>
      <div>
        <nav>
          <Link to="/">Home</Link>
          <Link to="/about">About</Link>
        </nav>
        <Route path="/" exact component={Home} />
        <Route path="/about" component={About} />
      </div>
    </Router>
  );
}
```

**Machine Coding Task:**  
Write a simple React Router setup with two routes: `/home` and `/contact`. Create corresponding components and navigation links.

```jsx
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

function Home() {
  return <h2>Welcome to the Home Page</h2>;
}

function Contact() {
  return <h2>Contact Us at: contact@example.com</h2>;
}

function App() {
  return (
    <Router>
      <div>
        <nav>
          <Link to="/home">Home</Link>
          <Link to="/contact">Contact</Link>
        </nav>
        <Route path="/home" exact component={Home} />
        <Route path="/contact" component={Contact} />
      </div>
    </Router>
  );
}
```

---

### 10.2-how-do-you-use-dynamic-routes-with-react-router

**Conceptual Answer:**  
**Dynamic routing** allows you to use URL parameters to pass dynamic data to a component. This is especially useful for routes that display data based on the ID or name in the URL.

To define a dynamic route, use a colon `:` followed by the parameter name in the route path. You can access the parameter via the **`match`** object in the component props.

**Example:**

```jsx
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

function UserProfile({ match }) {
  return <h2>User Profile of {match.params.username}</h2>;
}

function App() {
  return (
    <Router>
      <div>
        <nav>
          <Link to="/user/john">John's Profile</Link>
          <Link to="/user/jane">Jane's Profile</Link>
        </nav>
        <Route path="/user/:username" component={UserProfile} />
      </div>
    </Router>
  );
}
```

In this example, the route `/user/:username` dynamically renders the `UserProfile` component based on the `username` parameter passed in the URL.

**Machine Coding Task:**  
Write a component that dynamically renders user profiles based on the username in the URL (`/user/:username`), and displays the username on the page.

```jsx
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

function UserProfile({ match }) {
  return <h2>Profile of {match.params.username}</h2>;
}

function App() {
  return (
    <Router>
      <div>
        <nav>
          <Link to="/user/alice">Alice's Profile</Link>
          <Link to="/user/bob">Bob's Profile</Link>
        </nav>
        <Route path="/user/:username" component={UserProfile} />
      </div>
    </Router>
  );
}
```

---

### 10.3-how-do-you-implement-nested-routes-in-react-router

**Conceptual Answer:**  
**Nested routes** in React Router allow you to create a hierarchical route structure where a parent route renders child routes inside it. This is useful when you want to display a part of the UI based on sub-routes.

To implement nested routes:

1. Define a parent route with a child route inside it.
    
2. Use **`<Route>`** to specify the nested paths, and render the child components accordingly.
    

**Example:**

```jsx
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

function Dashboard() {
  return (
    <div>
      <h2>Dashboard</h2>
      <nav>
        <Link to="/dashboard/overview">Overview</Link>
        <Link to="/dashboard/stats">Stats</Link>
      </nav>
      <Route path="/dashboard/overview" component={Overview} />
      <Route path="/dashboard/stats" component={Stats} />
    </div>
  );
}

function Overview() {
  return <h3>Overview Page</h3>;
}

function Stats() {
  return <h3>Stats Page</h3>;
}

function App() {
  return (
    <Router>
      <Route path="/dashboard" component={Dashboard} />
    </Router>
  );
}
```

In this example, the `Dashboard` component renders either the `Overview` or `Stats` component based on the nested routes.

**Machine Coding Task:**  
Write a component with nested routes where the parent route `/dashboard` has two child routes: `/dashboard/settings` and `/dashboard/profile`.

```jsx
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

function Settings() {
  return <h3>Settings Page</h3>;
}

function Profile() {
  return <h3>Profile Page</h3>;
}

function Dashboard() {
  return (
    <div>
      <h2>Dashboard</h2>
      <nav>
        <Link to="/dashboard/settings">Settings</Link>
        <Link to="/dashboard/profile">Profile</Link>
      </nav>
      <Route path="/dashboard/settings" component={Settings} />
      <Route path="/dashboard/profile" component={Profile} />
    </div>
  );
}

function App() {
  return (
    <Router>
      <Route path="/dashboard" component={Dashboard} />
    </Router>
  );
}
```

---

### 10.4-how-do-you-handle-redirects-in-react-router

**Conceptual Answer:**  
In React Router, you can handle **redirects** using the **`Redirect`** component or the **`useHistory`** hook.

1. **`Redirect` Component**: Automatically navigates to a new route when rendered.
    
2. **`useHistory` Hook**: Programmatically navigates to a new route from a function or event handler.
    

**Example with `Redirect`:**

```jsx
import { BrowserRouter as Router, Route, Redirect, Link } from 'react-router-dom';

function Home() {
  return <h2>Home Page</h2>;
}

function App() {
  const loggedIn = false;

  return (
    <Router>
      <div>
        {loggedIn ? <Link to="/home">Go to Home</Link> : <Redirect to="/login" />}
        <Route path="/home" component={Home} />
        <Route path="/login" component={() => <h2>Login Page</h2>} />
      </div>
    </Router>
  );
}
```

In this example, if `loggedIn` is `false`, the user will be redirected to the `/login` page.

**Machine Coding Task:**  
Write a component that redirects to a "Not Found" page if the user attempts to visit a non-existent route.

```jsx
import { BrowserRouter as Router, Route, Redirect } from 'react-router-dom';

function NotFound() {
  return <h2>404 - Page Not Found</h2>;
}

function App() {
  return (
    <Router>
      <Route path="/not-found" component={NotFound} />
      <Redirect from="*" to="/not-found" />
    </Router>
  );
}
```

---

# 11-state-management-redux-mobx-RTK

### 11.1-what-is-redux-can-you-explain-its-core-principles-actions-reducers-store

**Conceptual Answer:**  
**Redux** is a state management library that helps manage the application state in a predictable way. It works on the principles of:

1. **Single Source of Truth**: The application state is stored in a single object called the **store**.
    
2. **State is Read-Only**: The only way to change the state is by dispatching **actions**.
    
3. **Changes are Made with Pure Functions**: Changes to the state are made using **reducers**, which are pure functions that take the current state and an action, and return the new state.
    

**Core Concepts:**

- **Actions**: Plain JavaScript objects that describe what happened in the app.
    
- **Reducers**: Functions that specify how the state changes in response to an action.
    
- **Store**: The object that holds the application state.
    

**Example:**

```javascript
// Action
const increment = () => ({ type: 'INCREMENT' });

// Reducer
const counter = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    default:
      return state;
  }
};

// Store
import { createStore } from 'redux';
const store = createStore(counter);
store.dispatch(increment());
console.log(store.getState()); // 1
```

---

### 11.2-how-do-you-connect-redux-to-a-react-component-can-you-show-an-example

**Conceptual Answer:**  
To connect Redux to a React component, use the `connect` function from the `react-redux` library. This function connects the component to the Redux store, allowing it to access state and dispatch actions.

**Steps to connect Redux to a React component:**

1. **`mapStateToProps`**: Maps the Redux state to the component's props.
    
2. **`mapDispatchToProps`**: Maps dispatchable actions to the component's props.
    

**Example:**

```javascript
import { connect } from 'react-redux';

// React Component
function Counter({ count, increment }) {
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

// Map state to props
const mapStateToProps = (state) => ({ count: state });

// Map dispatch to props
const mapDispatchToProps = (dispatch) => ({
  increment: () => dispatch({ type: 'INCREMENT' }),
});

// Connect component to Redux store
export default connect(mapStateToProps, mapDispatchToProps)(Counter);
```

---

### 11.3-what-are-the-differences-between-local-state-management-and-using-redux-for-state-management

**Conceptual Answer:**

1. **Local State**:
    
    - Managed within a single component.
        
    - Best suited for handling UI-specific state (e.g., form input, visibility toggles).
        
    - Does not need to be shared across multiple components.
        
2. **Redux**:
    
    - Global state that can be accessed and modified by any component connected to the Redux store.
        
    - Suitable for shared, application-wide state (e.g., user authentication, settings, data fetched from APIs).
        
    - Redux is often used when state needs to be shared between deeply nested or distant components.
        

**Example:**

- **Local State** (with `useState`):
    
    ```jsx
    function MyComponent() {
      const [counter, setCounter] = useState(0);
      return <button onClick={() => setCounter(counter + 1)}>{counter}</button>;
    }
    ```
    
- **Redux State**:
    
    ```javascript
    // Action
    const increment = () => ({ type: 'INCREMENT' });
    
    // Reducer
    const counterReducer = (state = 0, action) => {
      switch (action.type) {
        case 'INCREMENT':
          return state + 1;
        default:
          return state;
      }
    };
    ```
    

---

### 11.4-how-do-you-use-redux-thunk-or-redux-saga-for-handling-asynchronous-actions

**Conceptual Answer:**

1. **Redux Thunk**:
    
    - Middleware that allows action creators to return functions instead of action objects. This is useful for handling asynchronous actions like API calls.
        
    - You can dispatch actions inside the returned function.
        
    
    **Example with Redux Thunk:**
    
    ```javascript
    const fetchData = () => {
      return (dispatch) => {
        fetch('/api/data')
          .then((response) => response.json())
          .then((data) => {
            dispatch({ type: 'FETCH_SUCCESS', payload: data });
          });
      };
    };
    ```
    
2. **Redux Saga**:
    
    - A middleware that uses generators to handle side effects (asynchronous actions) more effectively than thunks.
        
    - Allows better handling of complex side effects, like waiting for multiple actions.
        
    
    **Example with Redux Saga:**
    
    ```javascript
    function* fetchDataSaga() {
      const response = yield call(fetch, '/api/data');
      const data = yield response.json();
      yield put({ type: 'FETCH_SUCCESS', payload: data });
    }
    
    function* rootSaga() {
      yield takeEvery('FETCH_REQUEST', fetchDataSaga);
    }
    ```
    

---

### 11.5-what-is-redux-toolkit-what-are-its-advantages-over-regular-redux

**Conceptual Answer:**  
**Redux Toolkit (RTK)** is a library that provides utilities to simplify the Redux setup and development process. It helps to reduce boilerplate code and improves the overall experience of using Redux.

**Advantages of Redux Toolkit:**

1. **Simplified Redux setup**: RTK provides a function `configureStore` that sets up the store with good defaults and simplifies store creation.
    
2. **`createSlice`**: Simplifies defining actions and reducers in one step.
    
3. **Built-in Thunks**: `createAsyncThunk` simplifies handling async actions, avoiding manually setting up thunks.
    
4. **Redux DevTools**: Automatically integrates with Redux DevTools for debugging.
    

**Example:**

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: (state) => state + 1,
    decrement: (state) => state - 1,
  },
});

const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
  },
});

export const { increment, decrement } = counterSlice.actions;
```

---

### 11.6-what-is-createasyncthunk-and-how-do-you-use-it-for-handling-asynchronous-actions

**Conceptual Answer:**  
`createAsyncThunk` is a function provided by Redux Toolkit that simplifies handling asynchronous actions in Redux. It allows you to define an action that automatically dispatches three action types:

1. **Pending**: Fired when the async operation starts.
    
2. **Fulfilled**: Fired when the async operation succeeds.
    
3. **Rejected**: Fired when the async operation fails.
    

**Example:**

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

export const fetchData = createAsyncThunk(
  'data/fetchData',
  async (arg, thunkAPI) => {
    const response = await fetch('/api/data');
    return response.json();
  }
);

const dataSlice = createSlice({
  name: 'data',
  initialState: { data: [], status: 'idle' },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchData.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchData.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload;
      })
      .addCase(fetchData.rejected, (state) => {
        state.status = 'failed';
      });
  },
});

const store = configureStore({
  reducer: {
    data: dataSlice.reducer,
  },
});
```

---

### 11.7-what-are-the-benefits-of-using-redux-toolkit-over-vanilla-redux

**Conceptual Answer:**  
Using **Redux Toolkit (RTK)** over traditional Redux has the following benefits:

1. **Less Boilerplate**: RTK reduces the need for action types, action creators, and reducers by combining them into slices.
    
2. **Simplified Async Handling**: `createAsyncThunk` and `createSlice` provide a more streamlined approach for handling asynchronous logic and action creators.
    
3. **Good Defaults**: RTK automatically includes Redux DevTools and sets up the store with best practices, such as middleware configuration.
    
4. **Improved Developer Experience**: RTK improves code readability, reduces potential errors, and speeds up development with utilities like `configureStore` and `createSlice`.
    

---

This concludes **Topic 11: State Management (Redux, MobX, etc.)** and **Redux Toolkit (RTK)**. Let me know if you'd like to proceed with the next topic!

# 12-error-boundaries

### 12.1-what-are-error-boundaries-in-react-why-are-they-important

**Conceptual Answer:**  
**Error boundaries** are React components that catch JavaScript errors anywhere in the component tree, log those errors, and display a fallback UI instead of crashing the entire component tree. They are important because they help keep the application running smoothly even when parts of the UI fail, preventing the entire app from crashing.

**Key Points about Error Boundaries:**

- **Catching errors**: Error boundaries catch errors in their child components.
    
- **Logging errors**: You can log the errors for debugging purposes (e.g., with an error reporting service).
    
- **Fallback UI**: Error boundaries can render a fallback UI when an error occurs.
    

**Example:**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({ errorInfo });
    // Log error to an error reporting service, etc.
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong.</h2>;
    }

    return this.props.children; 
  }
}

function App() {
  return (
    <ErrorBoundary>
      <MyComponent />
    </ErrorBoundary>
  );
}
```

---

### 12.2-how-do-you-implement-an-error-boundary-in-react

**Conceptual Answer:**  
To implement an error boundary in React, you need to create a class component with two specific lifecycle methods:

1. **`getDerivedStateFromError()`**: Called when an error is thrown, and it updates the state so the UI can display the fallback UI.
    
2. **`componentDidCatch()`**: Called when an error is thrown, and it allows you to log error information (e.g., to an error reporting service).
    

**Example:**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.log("Logged error:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong!</h2>;
    }
    return this.props.children;
  }
}
```

**Machine Coding Task:**  
Write an error boundary component and wrap it around a component that may throw an error (e.g., an invalid operation).

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.log('Error caught:', error);
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong</h2>;
    }
    return this.props.children;
  }
}

function BuggyComponent() {
  throw new Error("Error in the component!");
}

function App() {
  return (
    <ErrorBoundary>
      <BuggyComponent />
    </ErrorBoundary>
  );
}
```

---

### 12.3-what-is-the-difference-between-try-catch-and-error-boundaries-in-react

**Conceptual Answer:**

- **`try/catch`**: A JavaScript mechanism for handling errors synchronously within a block of code. It’s used for errors that occur during normal execution, such as network requests or synchronous code execution.
    
- **Error Boundaries**: React’s mechanism for handling errors in components, which works asynchronously by catching errors in any component tree rendered inside the boundary. It’s used to catch errors in rendering, lifecycle methods, and event handlers.
    

**Differences:**

- `try/catch` is used within JavaScript code, while error boundaries are used to catch errors during the React component lifecycle.
    
- `try/catch` doesn’t provide fallback UI, but error boundaries allow you to display fallback UI when an error occurs.
    

**Example (try/catch):**

```javascript
try {
  const result = someFunctionThatMightThrow();
} catch (error) {
  console.error('Error occurred:', error);
}
```

**Example (Error Boundary):**

```jsx
class MyErrorBoundary extends React.Component {
  // Same as previous error boundary example
}
```

**Machine Coding Task:**  
Write a component that throws an error in a lifecycle method (`componentDidMount`) and wrap it with an error boundary to display a fallback UI.

```jsx
class BuggyComponent extends React.Component {
  componentDidMount() {
    throw new Error("Error during component mount!");
  }

  render() {
    return <div>Component Loaded</div>;
  }
}

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong!</h2>;
    }
    return this.props.children;
  }
}

function App() {
  return (
    <ErrorBoundary>
      <BuggyComponent />
    </ErrorBoundary>
  );
}
```

---

# 13-performance-optimization

### 13.1-how-do-you-prevent-unnecessary-re-renders-in-react

**Conceptual Answer:**  
Unnecessary re-renders in React can lead to performance issues, especially in large applications. To prevent these, you can use several techniques:

1. **`React.memo`**:
    
    - A higher-order component that prevents re-renders of a component if its props haven’t changed.
        
    - Works for functional components.
        
    
    **Example:**
    
    ```jsx
    const MyComponent = React.memo(function MyComponent({ name }) {
      return <div>{name}</div>;
    });
    ```
    
2. **`shouldComponentUpdate`**:
    
    - In class components, you can override `shouldComponentUpdate` to return `false` if the component doesn’t need to re-render.
        
    
    **Example:**
    
    ```jsx
    class MyComponent extends React.Component {
      shouldComponentUpdate(nextProps, nextState) {
        return nextProps.name !== this.props.name;
      }
      render() {
        return <div>{this.props.name}</div>;
      }
    }
    ```
    
3. **`useMemo` and `useCallback`**:
    
    - Use `useMemo` to memoize the result of expensive calculations.
        
    - Use `useCallback` to memoize functions, preventing unnecessary re-creations on every render.
        
    
    **Example with `useMemo`:**
    
    ```jsx
    const expensiveCalculation = useMemo(() => computeExpensiveValue(a, b), [a, b]);
    ```
    
    **Example with `useCallback`:**
    
    ```jsx
    const memoizedFunction = useCallback(() => {
      doSomething(a, b);
    }, [a, b]);
    ```
    

**Machine Coding Task:**  
Write a component that uses `React.memo` to prevent re-renders when props haven't changed.

```jsx
const Counter = React.memo(({ count }) => {
  console.log('Rendering Counter');
  return <p>Count: {count}</p>;
});

function App() {
  const [count, setCount] = useState(0);
  const [otherState, setOtherState] = useState(0);

  return (
    <div>
      <Counter count={count} />
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <button onClick={() => setOtherState(otherState + 1)}>Increment Other</button>
    </div>
  );
}
```

---

### 13.2-what-is-reactmemo-and-when-would-you-use-it

**Conceptual Answer:**  
**`React.memo`** is a higher-order component that memoizes the rendered output of a component. If the props of the component don’t change, it skips the re-rendering process, improving performance in functional components.

- **When to use `React.memo`**:
    
    - When your component renders the same output for the same props, and you want to avoid unnecessary renders.
        
    - Useful for components that receive complex objects as props or components with expensive calculations.
        

**Example:**

```jsx
const MyComponent = React.memo(function MyComponent({ name }) {
  return <div>{name}</div>;
});
```

**Machine Coding Task:**  
Write a functional component that only re-renders when its `count` prop changes, using `React.memo`.

```jsx
const CountDisplay = React.memo(function CountDisplay({ count }) {
  return <div>Count: {count}</div>;
});

function App() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('John');

  return (
    <div>
      <CountDisplay count={count} />
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <button onClick={() => setName(name === 'John' ? 'Doe' : 'John')}>Change Name</button>
    </div>
  );
}
```

---

### 13.3-how-do-you-implement-code-splitting-and-lazy-loading-in-a-react-app

**Conceptual Answer:**  
**Code splitting** is a technique where you split the application bundle into smaller chunks that can be loaded as needed. **Lazy loading** allows you to load components only when they are needed, improving initial load performance.

To implement code splitting and lazy loading:

1. Use **`React.lazy()`** to dynamically import components.
    
2. Use **`Suspense`** to display a fallback (loading indicator) while the component is being loaded.
    

**Example:**

```jsx
import React, { Suspense, lazy } from 'react';

const LazyLoadedComponent = lazy(() => import('./LazyLoadedComponent'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <LazyLoadedComponent />
      </Suspense>
    </div>
  );
}
```

**Machine Coding Task:**  
Write a React app that lazily loads a component when a button is clicked.

```jsx
import React, { Suspense, lazy, useState } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  const [show, setShow] = useState(false);

  return (
    <div>
      <button onClick={() => setShow(true)}>Load Component</button>
      {show && (
        <Suspense fallback={<div>Loading...</div>}>
          <LazyComponent />
        </Suspense>
      )}
    </div>
  );
}
```

---

### 13.4-what-are-the-best-practices-for-optimizing-react-performance

**Conceptual Answer:**  
Here are some best practices for optimizing React performance:

1. **Use PureComponent/React.memo**: Ensure components only re-render when necessary by using `React.memo` for functional components and `PureComponent` for class components.
    
2. **Avoid Inline Functions and Objects**: Passing inline functions or objects as props causes re-renders on every render. Use `useCallback` or `useMemo` to memoize functions and objects.
    
3. **Implement Code Splitting and Lazy Loading**: Break your app into smaller chunks using `React.lazy()` and `Suspense`, ensuring only the necessary components are loaded at a time.
    
4. **Optimize Re-rendering**: Use **`shouldComponentUpdate`** or `React.memo` to prevent unnecessary re-renders.
    
5. **Debounce or Throttle** Expensive Operations: For operations like search input or window resizing, debounce or throttle them to avoid excessive re-renders.
    
6. **Avoid Heavy Computations in Render**: Keep computations outside the render method and only perform them if necessary.
    
7. **Optimize Context Usage**: Avoid excessive re-renders when using the Context API by splitting the context into smaller pieces.
    

**Machine Coding Task:**  
Write a component with a search input that debounces the search term to optimize performance.

```jsx
import { useState, useEffect } from 'react';

function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedTerm, setDebouncedTerm] = useState('');

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedTerm(searchTerm);
    }, 500);
    return () => clearTimeout(timer);
  }, [searchTerm]);

  const handleSearchChange = (e) => {
    setSearchTerm(e.target.value);
  };

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={handleSearchChange}
        placeholder="Search"
      />
      <p>Searching for: {debouncedTerm}</p>
    </div>
  );
}
```

---
# 14-testing
- 14.1-how-do-you-write-unit-tests-for-react-components-using-jest-and-react-testing-library
- 14.2-how-do-you-test-hooks-in-react
- 14.3-how-do-you-test-asynchronous-actions-and-side-effects-in-react-components

# 15-react-devtools

### 15.1-how-do-you-use-react-devtools-to-debug-react-applications

**Conceptual Answer:**  
**React DevTools** is a browser extension that allows you to inspect the React component tree and the state/props of each component. It helps with debugging by providing detailed insights into your React components, their hierarchy, and how they re-render.

**Steps to use React DevTools:**

1. Install the React DevTools extension for Chrome or Firefox.
    
2. Open your React app in the browser.
    
3. Open the browser's Developer Tools (F12) and navigate to the **React** tab.
    
4. You can inspect the component hierarchy, view the current state and props, and check if components are re-rendering unnecessarily.
    
5. You can also track component renders and log component updates.
    

**Example:**

1. Inspect the component tree and click on any component to view its state, props, and component tree.
    
2. Use the “Profiler” tab to see how often components are re-rendering and optimize accordingly.
    

**Machine Coding Task:**  
Install React DevTools and inspect the following component:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

Use React DevTools to inspect the component's state and props and track its re-renders.

---

### 15.2-what-are-some-key-features-of-react-devtools-for-performance-optimization

**Conceptual Answer:**  
React DevTools offers several features to help with performance optimization:

1. **Profiler Tab**: This shows how many times each component rendered and the duration of each render. It helps to identify which components are re-rendering too often and taking up unnecessary time.
    
    - **Record and Inspect**: You can record interactions and see how long each component takes to render.
        
    - **Highlight Updates**: When enabled, it highlights components that are being re-rendered in the UI.
        
2. **Component Inspector**: You can inspect the props and state of each component, which helps identify unnecessary changes or props passed down to components.
    
3. **Hooks**: The DevTools also let you inspect hooks and their current values (useState, useEffect, etc.) directly in the component tree.
    

**Machine Coding Task:**  
Use the Profiler tab of React DevTools to check the render performance of the following code:

```jsx
function ExpensiveComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('Expensive Component Mounted');
  }, []);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

Open the **Profiler** tab and record the component's renders. Try interacting with the component and check how many times it renders.

---

### 15.3-how-do-you-inspect-component-state-and-props-using-react-devtools

**Conceptual Answer:**  
React DevTools provides an easy way to inspect a component’s state and props.

1. **State**: You can inspect the internal state of a component by selecting it in the **Components** tab of React DevTools. Under "State," you will see the values of all the state variables for the selected component.
    
2. **Props**: In the same tab, you can also see the **props** passed to a component. This allows you to verify the data flowing from parent components to child components.
    

**Steps:**

1. Open React DevTools and navigate to the **Components** tab.
    
2. Click on the component you want to inspect.
    
3. On the right side, you'll see the **props** and **state**.
    
4. If you update the state in the component, React DevTools will show the updated state in real-time.
    

**Example:**

```jsx
function UserProfile({ name, age }) {
  const [status, setStatus] = useState('active');

  return (
    <div>
      <h1>{name}</h1>
      <p>{age}</p>
      <p>Status: {status}</p>
      <button onClick={() => setStatus('inactive')}>Deactivate</button>
    </div>
  );
}
```

In React DevTools, select the `UserProfile` component, and you’ll be able to see:

- **Props**: `name`, `age`
    
- **State**: `status`
    

**Machine Coding Task:**  
Inspect the `UserProfile` component above using React DevTools to check its props and state values. Track changes to `status` and how the state is updated when the button is clicked.

---
# 16-advanced-patterns
### 16.1-what-is-a-higher-order-component-hoc-and-how-do-you-use-it-in-react

**Conceptual Answer:**  
A **Higher-Order Component (HOC)** is a function that takes a component and returns a new component with additional props or functionality. HOCs allow you to reuse component logic across multiple components.

**Key Characteristics of HOCs:**

- HOCs are not components themselves, but functions that take components and return new components.
    
- They allow the addition of common logic like authentication checks, logging, or data fetching without repeating the logic in every component.
    

**Example:**

```jsx
// Higher-order component that adds a loading state to the wrapped component
function withLoading(WrappedComponent) {
  return function LoadingHOC(props) {
    if (props.isLoading) {
      return <p>Loading...</p>;
    }
    return <WrappedComponent {...props} />;
  };
}

// Regular component
function DataComponent({ data }) {
  return <div>{data}</div>;
}

// Wrapped component with loading behavior
const DataWithLoading = withLoading(DataComponent);

function App() {
  return <DataWithLoading isLoading={true} data="Here is the data" />;
}
```

In this example, `withLoading` is a HOC that adds loading functionality to any component it wraps.

**Machine Coding Task:**  
Write a higher-order component that adds authentication checks to a component. If the user is not authenticated, show a "Please log in" message.

```jsx
function withAuth(WrappedComponent) {
  return function AuthHOC(props) {
    if (!props.isAuthenticated) {
      return <p>Please log in</p>;
    }
    return <WrappedComponent {...props} />;
  };
}

function Dashboard() {
  return <h2>Welcome to the Dashboard</h2>;
}

const ProtectedDashboard = withAuth(Dashboard);

function App() {
  return <ProtectedDashboard isAuthenticated={false} />;
}
```

---

### 16.2-can-you-explain-render-props-and-give-an-example-of-how-to-use-them

**Conceptual Answer:**  
**Render props** is a pattern for sharing code between components in React. A component with a render prop takes a function as a prop, and the function is used to render content dynamically based on the component's state or props.

This pattern is often used for reusable logic, such as form handling, mouse tracking, or animations.

**Example:**

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('mousemove', handleMouseMove);

    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return render(position);
}

function App() {
  return (
    <MouseTracker
      render={({ x, y }) => <p>Mouse position: {x}, {y}</p>}
    />
  );
}
```

In this example, `MouseTracker` uses a render prop to pass the mouse position to its parent, and the parent uses the data to render it.

**Machine Coding Task:**  
Write a component using the render props pattern to track the window size (width and height) and display it.

```jsx
function WindowSizeTracker({ render }) {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };
    window.addEventListener('resize', handleResize);

    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return render(windowSize);
}

function App() {
  return (
    <WindowSizeTracker
      render={({ width, height }) => (
        <p>Window size: {width}x{height}</p>
      )}
    />
  );
}
```

---

### 16.3-what-are-compound-components-in-react-how-do-they-work

**Conceptual Answer:**  
**Compound components** is a pattern where multiple components work together to form a complex UI element. In this pattern, you use nested components that communicate with each other via context or shared state, allowing for highly flexible and reusable component designs.

- Compound components share an internal state and expose it to the child components through the parent.
    
- The parent component handles the logic, and the children render parts of the UI or accept different props.
    

**Example:**

```jsx
function Accordion({ children }) {
  const [openIndex, setOpenIndex] = useState(null);

  const handleClick = (index) => {
    setOpenIndex(openIndex === index ? null : index);
  };

  return (
    <div>
      {React.Children.map(children, (child, index) =>
        React.cloneElement(child, {
          index,
          isOpen: index === openIndex,
          onClick: handleClick,
        })
      )}
    </div>
  );
}

function AccordionItem({ index, isOpen, onClick, children }) {
  return (
    <div>
      <button onClick={() => onClick(index)}>Toggle</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}

function App() {
  return (
    <Accordion>
      <AccordionItem>Item 1 Content</AccordionItem>
      <AccordionItem>Item 2 Content</AccordionItem>
      <AccordionItem>Item 3 Content</AccordionItem>
    </Accordion>
  );
}
```

In this example, the `Accordion` component serves as the parent, and `AccordionItem` components are nested inside it. The parent manages the state, while the children display the content based on whether they are open.

**Machine Coding Task:**  
Write a compound component for a **Tabs** component, where the parent `Tabs` handles the active tab, and the child `Tab` components display their content based on the active tab.

```jsx
function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);

  const handleTabClick = (index) => {
    setActiveTab(index);
  };

  return (
    <div>
      {React.Children.map(children, (child, index) =>
        React.cloneElement(child, {
          index,
          activeTab,
          onClick: handleTabClick,
        })
      )}
    </div>
  );
}

function Tab({ index, activeTab, onClick, children }) {
  return (
    <div>
      <button onClick={() => onClick(index)}>Tab {index + 1}</button>
      {activeTab === index && <div>{children}</div>}
    </div>
  );
}

function App() {
  return (
    <Tabs>
      <Tab>Tab 1 Content</Tab>
      <Tab>Tab 2 Content</Tab>
      <Tab>Tab 3 Content</Tab>
    </Tabs>
  );
}
```

---
# 17-server-side-rendering-ssr

### 17.1-what-is-server-side-rendering-ssr-and-how-does-it-benefit-react-applications

**Conceptual Answer:**  
**Server-Side Rendering (SSR)** is the process of rendering a React application on the server rather than in the browser. In SSR, the server sends a fully rendered page to the client, and the JavaScript takes over on the client-side for subsequent interactions.

**Benefits of SSR in React:**

1. **Improved SEO**: Search engines can easily crawl the content of the application because it is rendered on the server and delivered as HTML.
    
2. **Faster First Load**: SSR reduces the time it takes to see the content on the page since the server sends a pre-rendered HTML page. This improves the perceived load time.
    
3. **Social Media Sharing**: Social media platforms often scrape the page content before sharing, so SSR ensures that the correct data (like titles and images) is available.
    

**How SSR works in React:**

1. The server renders the React components to HTML.
    
2. The HTML is sent to the client, where the React app is hydrated, meaning React takes over and adds interactivity.
    
3. Subsequent interactions are handled by React client-side.
    

**Example:**  
React can be used with frameworks like **Next.js** for SSR, which handles routing and SSR for you.

```javascript
// Example using Next.js
function HomePage() {
  return <h1>Hello, SSR with React!</h1>;
}

export default HomePage;
```

In this example, Next.js pre-renders the `HomePage` on the server before sending it to the client.

---

### 17.2-how-do-you-implement-ssr-with-react-what-tools-or-frameworks-would-you-use

**Conceptual Answer:**  
To implement SSR with React, you typically need a server that can render React components to HTML and send it to the client. **Node.js** is commonly used for SSR, and libraries like **Express** help serve the content. The process involves rendering the React components on the server, sending the rendered HTML to the client, and then hydrating the React app.

Here are the steps for SSR:

1. **Set up a Node.js server** (using Express or another framework).
    
2. **Use `react-dom/server`** to render the React components to HTML.
    
3. **Serve the rendered HTML** to the client.
    
4. **Hydrate** the React app on the client side.
    

**Example:**

```javascript
// Server-side rendering setup with Express and React
import express from 'express';
import React from 'react';
import ReactDOMServer from 'react-dom/server';
import App from './App';

const app = express();

app.get('/', (req, res) => {
  const content = ReactDOMServer.renderToString(<App />);
  res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <title>SSR with React</title>
    </head>
    <body>
      <div id="root">${content}</div>
      <script src="/bundle.js"></script>
    </body>
    </html>
  `);
});

app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
```

This example demonstrates rendering a React component (`App`) to a string on the server and sending the HTML to the client.

**Machine Coding Task:**  
Write a basic SSR setup using Express, React, and ReactDOMServer to render a simple "Hello, World!" message on the server.

```javascript
import express from 'express';
import React from 'react';
import ReactDOMServer from 'react-dom/server';

const app = express();

app.get('/', (req, res) => {
  const content = ReactDOMServer.renderToString(<h1>Hello, World from SSR!</h1>);
  res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <title>SSR Example</title>
    </head>
    <body>
      <div id="root">${content}</div>
    </body>
    </html>
  `);
});

app.listen(3000, () => {
  console.log('SSR app running at http://localhost:3000');
});
```

---

### 17.3-what-are-the-challenges-of-ssr-with-react-and-how-do-you-overcome-them

**Conceptual Answer:**  
While SSR provides several benefits, it also comes with its challenges:

1. **Initial Load Time**: Although SSR speeds up the first page load, the server still has to render the entire page on each request, which can add to the load time. To mitigate this, you can use techniques like **caching**.
    
2. **Client-Side Hydration**: After the server sends the rendered HTML, React has to "rehydrate" the page, attaching event listeners to the pre-rendered markup. This can cause a mismatch between the server-rendered HTML and the client-side React app. This issue can be minimized by ensuring that the server-rendered content matches the initial render of the React app.
    
3. **State Synchronization**: SSR doesn't inherently maintain client-side state, so synchronizing the initial state (e.g., user authentication) between server and client can be tricky. A common solution is to embed the state in the HTML and pass it to the React components during hydration.
    
4. **SEO and Dynamic Content**: Although SSR helps with SEO by providing fully rendered HTML to crawlers, handling dynamic content (e.g., real-time updates) can be more complex since it's difficult to re-render on the server after the initial request. This can be addressed by using **hydration** and client-side updates for dynamic content.
    
5. **Server-Side Performance**: SSR requires the server to perform heavy computation to render React components, which may put a strain on the server. This can be mitigated with **caching** strategies, such as caching the HTML output of frequently accessed pages or using **edge servers** to handle SSR at the network level.
    

**Solutions to overcome challenges:**

1. **Caching**: Use server-side caching for SSR to reduce the load time on subsequent requests.
    
2. **Code Splitting**: Use code splitting to reduce the amount of JavaScript that needs to be loaded on the client-side.
    
3. **Static Site Generation (SSG)**: Use tools like **Next.js** for generating static pages at build time, which can reduce server load.
    

**Example:**  
In **Next.js**, you can statically generate pages at build time, which is a combination of SSR and static site generation (SSG).

```javascript
// Example using Next.js
export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  return { props: { data } };
}

function Page({ data }) {
  return <div>{data}</div>;
}
```

**Machine Coding Task:**  
Write a basic SSR page in **Next.js** that fetches data from an API and renders it on the server. Set up the `getServerSideProps` function to fetch data from an API and render the page on the server.

```javascript
// pages/index.js in a Next.js project
export async function getServerSideProps() {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts');
  const posts = await res.json();
  return { props: { posts } };
}

function HomePage({ posts }) {
  return (
    <div>
      <h1>Server-Side Rendered Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}

export default HomePage;
```

---
# 18-typescript-with-react

### 18.1-how-do-you-type-react-components-with-typescript

**Conceptual Answer:**  
When using **TypeScript** with React, you can type your components by defining types for **props**, **state**, and **event handlers**. TypeScript provides type safety, which helps catch errors during development.

1. **Functional Components**: For functional components, you define prop types using an interface or type alias.
    
2. **Class Components**: For class components, you extend `React.Component` with types for both props and state.
    

**Example:**

1. **Functional Component with TypeScript**:
    

```tsx
interface MyComponentProps {
  name: string;
  age: number;
}

const MyComponent: React.FC<MyComponentProps> = ({ name, age }) => {
  return <div>{name} is {age} years old.</div>;
};

export default MyComponent;
```

In this example, `MyComponent` accepts `name` and `age` as props, and TypeScript ensures that these props are passed correctly.

2. **Class Component with TypeScript**:
    

```tsx
interface MyComponentProps {
  name: string;
  age: number;
}

interface MyComponentState {
  isActive: boolean;
}

class MyComponent extends React.Component<MyComponentProps, MyComponentState> {
  state: MyComponentState = {
    isActive: true,
  };

  render() {
    const { name, age } = this.props;
    return <div>{name} is {age} years old and is {this.state.isActive ? 'active' : 'inactive'}.</div>;
  }
}

export default MyComponent;
```

---

### 18.2-how-do-you-type-hooks-like-usestate-and-useeffect-in-typescript

**Conceptual Answer:**  
TypeScript allows you to type hooks like `useState` and `useEffect` in React, which improves code completion and error checking.

1. **`useState`**: The type of state can be explicitly set using TypeScript. If you don’t provide a type, TypeScript will infer it from the initial state value.
    
    **Example:**
    
    ```tsx
    const [count, setCount] = useState<number>(0);
    const [name, setName] = useState<string>('John');
    ```
    
2. **`useEffect`**: The `useEffect` hook itself doesn’t have a return value, but if you're performing asynchronous operations inside it (like fetching data), you can define the types of the data being fetched.
    
    **Example:**
    
    ```tsx
    const [data, setData] = useState<string | null>(null);
    
    useEffect(() => {
      const fetchData = async () => {
        const response = await fetch('https://api.example.com/data');
        const result: string = await response.json();
        setData(result);
      };
    
      fetchData();
    }, []);
    ```
    

**Machine Coding Task:**  
Write a component that uses `useState` to manage a count (number) and `useEffect` to fetch some data from an API and display it.

```tsx
import React, { useState, useEffect } from 'react';

const DataFetcher: React.FC = () => {
  const [data, setData] = useState<string | null>(null);
  const [count, setCount] = useState<number>(0);

  useEffect(() => {
    const fetchData = async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
      const result = await response.json();
      setData(result.title);
    };

    fetchData();
  }, []);

  return (
    <div>
      <p>{data}</p>
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <p>Count: {count}</p>
    </div>
  );
};

export default DataFetcher;
```

---

### 18.3-how-do-you-type-props-and-events-in-react-with-typescript

**Conceptual Answer:**  
In TypeScript, you can type both **props** and **events** to ensure type safety in your React components.

1. **Typing Props**: You can define types for props using an interface or type alias.
    
    **Example:**
    
    ```tsx
    interface ButtonProps {
      label: string;
      onClick: () => void;
    }
    
    const Button: React.FC<ButtonProps> = ({ label, onClick }) => {
      return <button onClick={onClick}>{label}</button>;
    };
    ```
    
2. **Typing Events**: TypeScript provides built-in types for various events such as `React.MouseEvent`, `React.ChangeEvent`, etc.
    
    **Example for typing an `onClick` event:**
    
    ```tsx
    const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
      console.log('Button clicked');
    };
    ```
    

**Machine Coding Task:**  
Write a component that accepts a `label` prop and triggers an `onClick` event handler, typed appropriately.

```tsx
import React from 'react';

interface ButtonProps {
  label: string;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
}

const Button: React.FC<ButtonProps> = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};

const App: React.FC = () => {
  const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    console.log('Button clicked!');
  };

  return <Button label="Click Me" onClick={handleClick} />;
};

export default App;
```

---

### 18.4-what-are-the-benefits-of-using-typescript-with-react-and-how-does-it-improve-development

**Conceptual Answer:**  
Using **TypeScript** with **React** provides several benefits, including:

1. **Static Type Checking**: TypeScript provides static type checking, which helps catch errors at compile-time rather than runtime, reducing bugs and improving code quality.
    
2. **Enhanced Code Completion**: TypeScript provides better IntelliSense and code completion in IDEs, making development more efficient and helping you catch mistakes early.
    
3. **Improved Refactoring**: TypeScript makes refactoring easier and safer by enforcing type constraints.
    
4. **Better Documentation**: Types serve as a form of documentation, making the codebase easier to understand and maintain.
    
5. **Integration with IDEs**: TypeScript works well with popular IDEs, offering features like auto-completion, type hints, and error checking.
    

**Example:**

```tsx
interface UserProps {
  name: string;
  age: number;
}

const UserProfile: React.FC<UserProps> = ({ name, age }) => {
  return (
    <div>
      <h1>{name}</h1>
      <p>Age: {age}</p>
    </div>
  );
};

// TypeScript ensures that the props are correctly passed
<UserProfile name="John Doe" age={30} />;
```

---

