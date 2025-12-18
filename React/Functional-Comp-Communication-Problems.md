# Table-of-Contents

<!-- toc -->

- [**1-Parent-to-Child-Communication-(Passing-Props)**](#1-parent-to-child-communication-passing-props)
- [**2-Child-to-Parent-Communication-(Using-Callback-Functions)**](#2-child-to-parent-communication-using-callback-functions)
- [**3-Child-to-Child-Communication-(via-Parent-Lifting-State-Up)**](#3-child-to-child-communication-via-parent-lifting-state-up)
- [**4-Passing-Functions-Down-the-Component-Tree**](#4-passing-functions-down-the-component-tree)
- [**5-Passing-Data-Through-Multiple-Layers-(Props-Drilling)**](#5-passing-data-through-multiple-layers-props-drilling)
- [**6-Event-Handlers-and-Prop-Forwarding**](#6-event-handlers-and-prop-forwarding)
- [**7-Functional-Composition-(Using-Higher-Order-Components)**](#7-functional-composition-using-higher-order-components)
- [**8-Sibling-Communication-via-Parent-(Lifting-State-Up)**](#8-sibling-communication-via-parent-lifting-state-up)
- [**9-Managing-State-with-Multiple-Children-(Complex-State-Sharing)**](#9-managing-state-with-multiple-children-complex-state-sharing)
- [**10-Custom-Hooks-for-Component-Communication**](#10-custom-hooks-for-component-communication)
- [**11-Compound-Components-Pattern-(Shared-State)**](#11-compound-components-pattern-shared-state)
- [**12-Context-like-Pattern-without-Using-the-Context-API**](#12-context-like-pattern-without-using-the-context-api)
- [**13-Forwarding-Refs-to-Child-Components**](#13-forwarding-refs-to-child-components)

<!-- tocstop -->

---
### **1-Parent-to-Child-Communication-(Passing-Props)**

**Problem Statement**:  
Create a **Parent** component that passes a `message` string as a prop to a **Child** component. The **Child** component should display the passed `message` inside a `<div>`.

---

### **2-Child-to-Parent-Communication-(Using-Callback-Functions)**

**Problem Statement**:  
Create a **Parent** component that passes a callback function `handleMessage` to a **Child** component. The **Child** component should call this function with a message (e.g., `"Hello from Child!"`) when a button is clicked. The **Parent** should log the received message.

---

### **3-Child-to-Child-Communication-(via-Parent-Lifting-State-Up)**

**Problem Statement**:  
Create a **Parent** component that maintains a `message` state and passes it to two **Child** components. **Child A** should be able to send a message to the parent when a button is clicked, and **Child B** should display the updated message from the parent.

---

### **4-Passing-Functions-Down-the-Component-Tree**

**Problem Statement**:  
Create a **Parent** component that has a state variable `count`. Pass a function to **Child A** that increments `count`. Display the `count` in **Child B**. Each time the button in **Child A** is clicked, the count should increment in **Parent**, and the updated count should be displayed in **Child B**.

---

### **5-Passing-Data-Through-Multiple-Layers-(Props-Drilling)**

**Problem Statement**:  
Create a **GrandParent** component that passes a `name` prop down to **Child** through **Parent**. **Child** should display the `name` it receives from the parent.

---

### **6-Event-Handlers-and-Prop-Forwarding**

**Problem Statement**:  
Create a **Parent** component that passes an event handler `handleClick` to **Child**. The **Child** component should have a button that, when clicked, invokes `handleClick` in the **Parent**. The **Parent** should log a message when the button in **Child** is clicked.

---

### **7-Functional-Composition-(Using-Higher-Order-Components)**

**Problem Statement**:  
Create a **Higher-Order Component (HOC)** `withTitle` that adds a title "Enhanced Component" above any component passed to it. Use the HOC to enhance a **BaseComponent** that displays some content, and render it inside **App**.

---

### **8-Sibling-Communication-via-Parent-(Lifting-State-Up)**

**Problem Statement**:  
Create a **Parent** component that manages a `count` state and passes the `count` to **Child A** and a function `incrementCount` to **Child B**. **Child A** should display the current count, and **Child B** should have a button to increment the count.

---

### **9-Managing-State-with-Multiple-Children-(Complex-State-Sharing)**

**Problem Statement**:  
Create a **Parent** component that manages a shared state `items`. **Child A** should have a button that adds an item to the `items` array. **Child B** should display the updated `items` array from the **Parent**.

---

### **10-Custom-Hooks-for-Component-Communication**

**Problem Statement**:  
Create a custom hook `useCounter` that manages a `counter` state and provides an `increment` function to update the counter. The **Parent** component should use `useCounter` and pass the `increment` function to **Child A** and the `counter` state to **Child B**.

---

### **11-Compound-Components-Pattern-(Shared-State)**

**Problem Statement**:  
Create an **Accordion** component that manages the state of which **Panel** is currently open. The **Accordion** should pass the open panel state and a function to toggle the panel to each **Panel** component. Only one panel should be open at a time.

---

### **12-Context-like-Pattern-without-Using-the-Context-API**

**Problem Statement**:  
Create a custom state management solution where multiple child components can access and update the same state from the parent, without using the Context API. The **Parent** should manage the shared state (e.g., `counter`) and provide functions to access and update it. The **Child A** component should display the current counter, and **Child B** should have a button to increment it.

---

### **13-Forwarding-Refs-to-Child-Components**

**Problem Statement**:  
Use **`React.forwardRef`** to forward a `ref` from the **Parent** component to a **Child** component. The **Child** component should have an `input` field, and the **Parent** should be able to focus the `input` field when a button is clicked in the **Parent**.

---
