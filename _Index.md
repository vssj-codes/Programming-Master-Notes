# Table-of-Contents

<!-- toc -->

- [**Table-of-Index**](#table-of-index)
      - [**Topic 1: JSX and Rendering**](#topic-1-jsx-and-rendering)

<!-- tocstop -->

---


# **Table-of-Index**

1. **JSX and Rendering**
    
    - 1.1 What is JSX? How is it different from HTML?
        
    - 1.2 How is JSX transpiled to JavaScript? Can you demonstrate this with code?
        
    - 1.3 Explain how React renders elements to the DOM. How does the virtual DOM fit into this?
        
    - 1.4 How do you handle conditions in JSX?
        
2. **Components**
    
    - 2.1 What is the difference between functional and class components in React?
        
    - 2.2 Explain the component lifecycle. What are the methods in a class component lifecycle?
        
    - 2.3 How do you manage state and props in a component?
        
    - 2.4 How do you pass data from a parent to a child component using props?
        
3. **State and Props**
    
    - 3.1 What is the difference between state and props in React? Can you give an example?
        
    - 3.2 How do you lift state from a child component to a parent component?
        
    - 3.3 How do you pass functions as props to child components? Can you provide an example?
        
4. **Event Handling**
    
    - 4.1 How does event handling work in React? What are the differences from traditional DOM events?
        
    - 4.2 How do you bind event handlers in React? Can you show how to handle a click event in a class component vs a functional component?
        
    - 4.3 How do you handle controlled and uncontrolled components in forms?
        
5. **Conditional Rendering**
    
    - 5.1 How do you implement conditional rendering in React?
        
    - 5.2 How do you use `ternary` operators and logical operators for conditional rendering?
        
    - 5.3 Can you demonstrate conditional rendering using the `&&` operator in JSX?
        
6. **Lists and Keys**
    
    - 6.1 How do you render a list of items in React using `map`?
        
    - 6.2 Why is the `key` prop important when rendering lists in React? What can go wrong without it?
        
    - 6.3 How would you handle dynamic lists of components, and how can you optimize the re-rendering process?
        
7. **Forms and User Input**
    
    - 7.1 How do you handle forms in React? What’s the difference between controlled and uncontrolled components?
        
    - 7.2 How do you manage multiple inputs in a form using state?
        
    - 7.3 How would you implement form validation in React?
        
    - 7.4 Can you demonstrate how to handle form submission in a React app?
        
8. **Hooks**
    
    - 8.1 What is the `useState` hook and how do you use it in functional components?
        
    - 8.2 Explain the `useEffect` hook. How does it mimic lifecycle methods in class components?
        
    - 8.3 How do you use the `useContext` hook to share state globally?
        
    - 8.4 How do you create custom hooks in React? Can you show an example?
        
    - 8.5 How do you clean up side effects using the `useEffect` hook?
        
9. **Context API**
    
    - 9.1 What is the Context API in React? Why would you use it over prop drilling?
        
    - 9.2 How do you create and use a Context in React?
        
    - 9.3 How do you consume a context in a functional component?
        
10. **React Router**
    
    - 10.1 What is React Router and how do you set up basic routing in a React app?
        
    - 10.2 How do you use dynamic routes with React Router?
        
    - 10.3 How do you implement nested routes in React Router?
        
    - 10.4 How do you handle redirects in React Router?
        
11. **State Management (Redux, MobX, etc.)**
    
    - 11.1 What is Redux? Can you explain its core principles (Actions, Reducers, Store)?
        
    - 11.2 How do you connect Redux to a React component? Can you show an example?
        
    - 11.3 What are the differences between local state management and using Redux for state management?
        
    - 11.4 How do you use Redux Thunk or Redux Saga for handling asynchronous actions?
        
12. **Error Boundaries**
    
    - 12.1 What are error boundaries in React? Why are they important?
        
    - 12.2 How do you implement an error boundary in React?
        
    - 12.3 What is the difference between try-catch and error boundaries in React?
        
13. **Performance Optimization**
    
    - 13.1 How do you prevent unnecessary re-renders in React?
        
    - 13.2 What is React.memo and when would you use it?
        
    - 13.3 How do you implement code-splitting and lazy loading in a React app?
        
14. **Testing**
    
    - 14.1 How do you write unit tests for React components using Jest and React Testing Library?
        
    - 14.2 How do you test hooks in React?
        
    - 14.3 How do you test asynchronous actions and side effects in React components?
        
15. **React DevTools**
    
    - 15.1 How do you use React DevTools to debug React applications?
        
    - 15.2 What are some key features of React DevTools for performance optimization?
        
    - 15.3 How do you inspect component state and props using React DevTools?
        
16. **Advanced Patterns**
    
    - 16.1 What is a Higher-Order Component (HOC) and how do you use it in React?
        
    - 16.2 Can you explain render props and give an example of how to use them?
        
    - 16.3 What are compound components in React? How do they work?
        
17. **Server-Side Rendering (SSR)**
    
    - 17.1 What is Server-Side Rendering (SSR) and how does it benefit React applications?
        
    - 17.2 How do you implement SSR with React? What tools or frameworks would you use?
        
    - 17.3 What are the challenges of SSR with React, and how do you overcome them?
        
18. **TypeScript with React**
    
    - 18.1 How do you type React components with TypeScript?
        
    - 18.2 How do you type hooks like `useState` and `useEffect` in TypeScript?
        
    - 18.3 How do you type props and events in React with TypeScript?
        
    - 18.4 What are the benefits of using TypeScript with React, and how does it improve development?
        

---

#### **Topic 1: JSX and Rendering**

1. **What is JSX? How is it different from HTML?**
    
    **Conceptual Answer:**  
    JSX is a syntax extension for JavaScript that allows us to write HTML-like code inside JavaScript. It is not exactly HTML but gets compiled to JavaScript objects. The major difference between JSX and HTML is that JSX uses `className` instead of `class`, and events are written in camelCase (e.g., `onClick` instead of `onclick`).
    
    **Machine Coding Task:**  
    Write a React component that renders a `div` containing a `button`. The button should say “Click me”, and when clicked, it should show an alert that says “Hello, JSX!”
    
2. **How is JSX transpiled to JavaScript? Can you demonstrate this with code?**
    
    **Conceptual Answer:**  
    JSX is not valid JavaScript and needs to be transpiled. React uses Babel to transpile JSX into `React.createElement()` calls. For example, `<div>Hello, world!</div>` is transpiled to `React.createElement('div', null, 'Hello, world!')`.
    
    **Machine Coding Task:**  
    Write a simple JSX code in React and show how it gets transpiled into the JavaScript equivalent.
