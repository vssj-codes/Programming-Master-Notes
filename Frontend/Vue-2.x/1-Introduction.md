# Table-of-Contents

<!-- toc -->

- [1- What-Is-Vue-Js](#1--what-is-vue-js)
  * [Definition](#definition)
  * [Key Concepts](#key-concepts)
  * [Example](#example)
  * [Real-World Use Cases](#real-world-use-cases)
  * [Gotchas](#gotchas)
- [2-The-Progressive-Framework](#2-the-progressive-framework)
  * [Definition](#definition-1)
  * [Key Concepts](#key-concepts-1)
  * [Use Cases](#use-cases)
- [3-Getting-Started](#3-getting-started)
  * [Methods](#methods)
  * [Use Cases](#use-cases-1)
- [4-Declarative-Rendering](#4-declarative-rendering)
  * [Definition](#definition-2)
  * [Example](#example-1)
  * [Key Point](#key-point)
- [5-Conditionals-And-Loops](#5-conditionals-and-loops)
  * [Conditional Directives](#conditional-directives)
  * [Looping](#looping)
  * [Example](#example-2)
- [6-Handling-User-Input](#6-handling-user-input)
  * [Directive](#directive)
  * [Example](#example-3)
  * [Use Case](#use-case)
- [7-Composing-With-Components](#7-composing-with-components)
  * [Concept](#concept)
  * [Example](#example-4)
  * [Use Case](#use-case-1)

<!-- tocstop -->

---

## 1- What-Is-Vue-Js

### Definition

Vue.js is a **progressive JavaScript framework** for building **user interfaces**. It is designed from the ground up to be incrementally adoptable and focuses on the **view layer only**.

### Key Concepts

- Approachable for beginners, yet powerful for professionals.
    
- Component-based architecture.
    
- Declarative rendering using HTML-based templates.
    
- Reactivity system automatically updates the DOM when data changes.
    

### Example

```html
<div id="app">{{ message }}</div>

<script>
  new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue!'
    }
  });
</script>
```

### Real-World Use Cases

- Embed Vue in a single page or widget.
    
- Scale to complex applications using Vue Router and Vuex.
    

### Gotchas

- Vue is only reactive for properties defined at instantiation.
    
- Use `Vue.set()` to add reactive properties later.
    

---

## 2-The-Progressive-Framework

### Definition

Vue is called **progressive** because you can gradually adopt it. Start with a single component or element and scale up to a full-featured application.

### Key Concepts

- Lightweight core focused on the view layer.
    
- Companion libraries (`vue-router`, `vuex`) add advanced features.
    
- Allows integration into projects step-by-step.
    

### Use Cases

- Progressive enhancement of server-rendered pages.
    
- Replacement of jQuery for small interactions.
    
- Full SPA development.
    

---

## 3-Getting-Started

### Methods

1. **Via CDN**
    
    ```html
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    ```
    
2. **Via NPM**
    
    ```bash
    npm install vue
    ```
    
3. **Via Vue CLI**
    
    ```bash
    npm install -g @vue/cli
    vue create my-project
    cd my-project
    npm run serve
    ```
    

### Use Cases

- Use CDN for quick testing.
    
- Use NPM for module-based bundling.
    
- Use CLI for full SPA scaffolding.
    

---

## 4-Declarative-Rendering

### Definition

Vue uses declarative syntax to bind the DOM to the application state. When data changes, the DOM automatically updates.

### Example

```html
<div id="app">{{ message }}</div>

<script>
  new Vue({
    el: '#app',
    data: {
      message: 'Hello!'
    }
  });
</script>
```

### Key Point

The `message` variable is bound to the DOM using the `{{ }}` syntax and will update reactively.

---

## 5-Conditionals-And-Loops

### Conditional Directives

- `v-if`: Conditionally render an element.
    
- `v-else-if`, `v-else`: Chained conditionals.
    
- `v-show`: Toggle visibility (CSS `display`).
    

### Looping

- `v-for`: Render elements based on a list.
    

### Example

```html
<p v-if="seen">Now you see me</p>

<ul>
  <li v-for="item in items" :key="item.id">
    {{ item.text }}
  </li>
</ul>
```

---

## 6-Handling-User-Input

### Directive

- `v-on`: Bind event listeners.
    

### Example

```html
<button v-on:click="counter++">Add 1</button>

<script>
  new Vue({
    el: '#app',
    data: {
      counter: 0
    }
  });
</script>
```

### Use Case

Update state reactively in response to user actions like clicks, inputs, and keypresses.

---

## 7-Composing-With-Components

### Concept

Vue applications are composed of components — self-contained units of functionality and UI.

### Example

```js
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
});
```

```html
<todo-item
  v-for="item in groceryList"
  :key="item.id"
  :todo="item"
></todo-item>
```

### Use Case

Divide your app into small, reusable pieces like buttons, cards, or entire views.
