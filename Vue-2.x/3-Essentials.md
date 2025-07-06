# Table-of-Contents

<!-- toc -->

- [3-Essentials](#3-essentials)
  * [3.1-Instance](#31-instance)
    + [Definition](#definition)
    + [Options](#options)
    + [Example](#example)
  * [3.2-Template-Syntax](#32-template-syntax)
    + [Interpolation](#interpolation)
    + [Directives](#directives)
    + [Expressions](#expressions)
    + [Attributes](#attributes)
  * [3.3-Computed-Properties-And-Watchers](#33-computed-properties-and-watchers)
    + [Computed Properties](#computed-properties)
    + [Watchers](#watchers)
  * [3.4-Class-And-Style-Bindings](#34-class-and-style-bindings)
    + [Class Bindings](#class-bindings)
    + [Binding with computed property](#binding-with-computed-property)
    + [Style Bindings](#style-bindings)
  * [3.5-Conditional-Rendering](#35-conditional-rendering)
    + [Directives](#directives-1)
    + [Key Difference](#key-difference)
    + [Example](#example-1)
  * [3.6-List-Rendering](#36-list-rendering)
    + [`v-for` Syntax](#v-for-syntax)
    + [Accessing Index](#accessing-index)
    + [Objects](#objects)
  * [3.7-Event-Handling](#37-event-handling)
    + [Syntax](#syntax)
    + [Example](#example-2)
    + [Methods](#methods)
    + [Event Object](#event-object)
  * [3.8-Form-Input-Bindings](#38-form-input-bindings)
    + [`v-model` Directive](#v-model-directive)
    + [Input Types](#input-types)
    + [Checkbox & Radio](#checkbox--radio)
    + [Modifiers](#modifiers)
  * [3.9-Components-Basics](#39-components-basics)
    + [Registration](#registration)
    + [Props](#props)
    + [Template](#template)
    + [Parent Usage](#parent-usage)
    + [Data Must Be Function](#data-must-be-function)
    + [Important](#important)

<!-- tocstop -->

---

# 3-Essentials

## 3.1-Instance

### Definition

The root Vue instance is created using `new Vue()`. Every Vue application starts by creating a new Vue instance.

### Options

- `el`: Mounting point (CSS selector or DOM element).
    
- `data`: Reactive state.
    
- `methods`: Functions accessible from templates.
    
- `computed`: Cached expressions based on reactive state.
    
- `watch`: Watchers for reacting to data changes.
    
- `mounted`: Lifecycle hook called after DOM insertion.
    

### Example

```js
new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  },
  methods: {
    greet: function () {
      alert(this.message);
    }
  }
});
```

---

## 3.2-Template-Syntax

### Interpolation

- Text: `{{ message }}`
    
- HTML: `v-html="rawHtml"`
    

### Directives

- `v-bind`: Bind attribute or prop.
    
- `v-on`: Listen to events.
    
- Shorthands:
    
    - `:` for `v-bind`
        
    - `@` for `v-on`
        

### Expressions

- You can use JS expressions inside double curly braces.
    
    ```html
    {{ number + 1 }}
    ```
    

### Attributes

```html
<a v-bind:href="url">Link</a>
```

---

## 3.3-Computed-Properties-And-Watchers

### Computed Properties

- Cached based on their reactive dependencies.
    
- Re-evaluated only when a dependency changes.
    

```js
computed: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('');
  }
}
```

### Watchers

- Useful for asynchronous operations or expensive computations.
    

```js
watch: {
  question: function (newQ) {
    this.answer = 'Waiting...';
    this.getAnswer();
  }
}
```

---

## 3.4-Class-And-Style-Bindings

### Class Bindings

```html
<div :class="{ active: isActive }"></div>
```

### Binding with computed property

```js
computed: {
  classObject: function () {
    return {
      active: this.isActive,
      'text-danger': this.hasError
    };
  }
}
```

### Style Bindings

```html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

---

## 3.5-Conditional-Rendering

### Directives

- `v-if`, `v-else-if`, `v-else`
    
- `v-show`: Always renders element, toggles visibility with CSS
    

### Key Difference

- `v-if` is lazy; `v-show` is faster for frequent toggles.
    

### Example

```html
<p v-if="seen">Now you see me</p>
```

---

## 3.6-List-Rendering

### `v-for` Syntax

```html
<li v-for="item in items" :key="item.id">
  {{ item.text }}
</li>
```

### Accessing Index

```html
<li v-for="(item, index) in items">{{ index }} - {{ item }}</li>
```

### Objects

```html
<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }}: {{ value }}
</div>
```

---

## 3.7-Event-Handling

### Syntax

- `v-on:eventName="method"`
    
- Shorthand: `@click="method"`
    

### Example

```html
<button @click="increment">Add 1</button>
```

### Methods

```js
methods: {
  increment: function () {
    this.count++;
  }
}
```

### Event Object

```html
<button @click="doSomething($event)">Click me</button>
```

---

## 3.8-Form-Input-Bindings

### `v-model` Directive

- Two-way binding between form input and data.
    

### Input Types

```html
<input v-model="message" placeholder="edit me">
<textarea v-model="message"></textarea>
<select v-model="selected">
  <option>A</option>
  <option>B</option>
</select>
```

### Checkbox & Radio

```html
<input type="checkbox" v-model="checked">
<input type="radio" v-model="picked" value="One">
```

### Modifiers

- `.lazy`: Update on change instead of input
    
- `.number`: Cast input to number
    
- `.trim`: Trim whitespace
    

---

## 3.9-Components-Basics

### Registration

- Global: `Vue.component('my-comp', {...})`
    
- Local: Define in `components` option
    

### Props

- Passed from parent to child
    

```js
props: ['title']
```

### Template

```js
template: '<div>{{ title }}</div>'
```

### Parent Usage

```html
<my-comp title="Hello"></my-comp>
```

### Data Must Be Function

```js
data: function() {
  return {
    count: 0
  };
}
```

### Important

- Component names must be kebab-case in templates.
    
- Root component can only have one root element in the template.