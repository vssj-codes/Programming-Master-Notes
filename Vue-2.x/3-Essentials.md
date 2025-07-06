# Table-of-Contents

<!-- toc -->

- [3-Essentials](#3-essentials)
  * [3.1-Instance](#31-instance)
    + [3.1.1-Creating-a-Vue-instance](#311-creating-a-vue-instance)
    + [3.1.2-Root-element-el](#312-root-element-el)
    + [3.1.3-Data-option](#313-data-option)
    + [3.1.4-Methods](#314-methods)
    + [3.1.5-Lifecycle-hooks-overview](#315-lifecycle-hooks-overview)
  * [3.2-Template-Syntax](#32-template-syntax)
    + [3.2.1-Text-interpolation](#321-text-interpolation)
    + [3.2.2-Raw-HTML](#322-raw-html)
    + [3.2.3-Attribute-bindings](#323-attribute-bindings)
      - [3.2.3.1-v-bind](#3231-v-bind)
      - [3.2.3.2-Shorthand](#3232-shorthand)
    + [3.2.4-Event-bindings](#324-event-bindings)
      - [3.2.4.1-v-on](#3241-v-on)
      - [3.2.4.2-Shorthand](#3242-shorthand)
    + [3.2.5-JavaScript-expressions-in-templates](#325-javascript-expressions-in-templates)
  * [3.3-Directives-In-Depth](#33-directives-in-depth)
    + [3.3.1-Arguments](#331-arguments)
    + [3.3.2-Dynamic-arguments](#332-dynamic-arguments)
    + [3.3.3-Modifiers](#333-modifiers)
      - [3.3.3.1-Event-modifiers](#3331-event-modifiers)
      - [3.3.3.2-Model-modifiers](#3332-model-modifiers)
      - [3.3.3.3-Binding-modifiers](#3333-binding-modifiers)
  * [3.4-Computed-Properties-And-Watchers](#34-computed-properties-and-watchers)
    + [3.4.1-Computed-properties](#341-computed-properties)
    + [3.4.2-Methods-vs-Computed](#342-methods-vs-computed)
    + [3.4.3-Watchers](#343-watchers)
  * [3.5-Class-And-Style-Bindings](#35-class-and-style-bindings)
    + [3.5.1-Object-syntax-for-class](#351-object-syntax-for-class)
    + [3.5.2-Array-syntax-for-class](#352-array-syntax-for-class)
    + [3.5.3-Binding-inline-styles](#353-binding-inline-styles)
    + [3.5.4-Using-computed-properties-for-classes-styles](#354-using-computed-properties-for-classes-styles)
  * [3.6-Conditional-Rendering](#36-conditional-rendering)
    + [3.6.1-v-if-v-else-if-v-else](#361-v-if-v-else-if-v-else)
    + [3.6.2-v-show](#362-v-show)
    + [3.6.3-Key-differences](#363-key-differences)
    + [3.6.4-template-tag-with-conditionals](#364-template-tag-with-conditionals)
  * [3.7-List-Rendering](#37-list-rendering)
    + [3.7.1-v-for-with-arrays](#371-v-for-with-arrays)
    + [3.7.2-v-for-with-objects](#372-v-for-with-objects)
    + [3.7.3-v-for-with-index](#373-v-for-with-index)
    + [3.7.4-Key-binding](#374-key-binding)
    + [3.7.5-Nested-v-for](#375-nested-v-for)
    + [3.7.6-template-usage](#376-template-usage)
  * [3.8-Event-Handling](#38-event-handling)
    + [3.8.1-Listening-with-v-on](#381-listening-with-v-on)
    + [3.8.2-Method-handlers](#382-method-handlers)
    + [3.8.3-Inline-handlers](#383-inline-handlers)
    + [3.8.4-Event-object-access](#384-event-object-access)
    + [3.8.5-Event-modifiers](#385-event-modifiers)
    + [3.8.6-Key-modifiers](#386-key-modifiers)
    + [3.8.7-System-modifiers](#387-system-modifiers)
    + [3.8.8-Exact-modifier](#388-exact-modifier)
  * [3.9-Form-Input-Bindings](#39-form-input-bindings)
    + [3.9.1-v-model-basics](#391-v-model-basics)
    + [3.9.2-Input-types](#392-input-types)
    + [3.9.3-Multiple-checkboxes-select](#393-multiple-checkboxes-select)
    + [3.9.4-Modifiers](#394-modifiers)
  * [3.10-Components-Basics](#310-components-basics)
    + [3.10.1-Global-and-local-registration](#3101-global-and-local-registration)
    + [3.10.2-Props](#3102-props)
    + [3.10.3-Template-syntax-for-props](#3103-template-syntax-for-props)
    + [3.10.4-Data-as-a-function](#3104-data-as-a-function)
    + [3.10.5-Custom-events-with-$emit](#3105-custom-events-with-emit)
    + [3.10.6-Parent-child-communication](#3106-parent-child-communication)
    + [3.10.7-One-root-element-rule](#3107-one-root-element-rule)
    + [3.10.8-Reusability-and-encapsulation](#3108-reusability-and-encapsulation)

<!-- tocstop -->

---

# 3-Essentials

## 3.1-Instance

### 3.1.1-Creating-a-Vue-instance

- Use `new Vue({ ... })` to create a Vue root instance.
    
- This is the entry point for a Vue app.
    

```js
new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
});
```

### 3.1.2-Root-element-el

- The `el` option specifies the DOM element to mount the Vue instance.
    
- If omitted, you can mount later using `$mount()`.
    

### 3.1.3-Data-option

- `data` defines the reactive state object.
    
- Properties declared here become reactive and accessible in templates.
    

### 3.1.4-Methods

- Defined in the `methods` object.
    
- Can be invoked in the template using event bindings.
    
- Do not cache results (use computed for that).
    

### 3.1.5-Lifecycle-hooks-overview

- Common hooks:
    
    - `created`, `mounted`, `updated`, `destroyed`
        
- Use hooks to perform actions at specific stages.
    

---

## 3.2-Template-Syntax

### 3.2.1-Text-interpolation

- Use `{{ msg }}` to render text content.
    
- Only one expression per interpolation.
    

### 3.2.2-Raw-HTML

- Use `v-html="rawHtml"` to render raw HTML content.
    
- Be cautious—can lead to XSS if untrusted content is used.
    

### 3.2.3-Attribute-bindings

#### 3.2.3.1-v-bind

- Dynamically bind one or more attributes.
    
- Basic syntax: `v-bind:href="url"`
    

#### 3.2.3.2-Shorthand

- Use `:` as shorthand for `v-bind`.
    
- Example: `:href="url"`
    

### 3.2.4-Event-bindings

#### 3.2.4.1-v-on

- Attach event listeners: `v-on:click="doSomething"`
    

#### 3.2.4.2-Shorthand

- Use `@` as shorthand: `@click="doSomething"`
    

### 3.2.5-JavaScript-expressions-in-templates

- Allowed: simple expressions (e.g. `{{ number + 1 }}`)
    
- Not allowed: statements or complex logic
    



---

## 3.3-Directives-In-Depth

- Special attributes with `v-` prefix. It's job is to reactively apply side effects to the DOM when the value of its expression changes
    
- Examples: `v-model`, `v-if`, `v-for`, `v-show`, `v-bind`, `v-on`

### 3.3.1-Arguments

- Some directives take an argument: `v-bind:href`, `v-on:click`
    

### 3.3.2-Dynamic-arguments

- Use square brackets: `v-bind:[attribute]="url"`
    
- Attribute must resolve to a string.
    

### 3.3.3-Modifiers

#### 3.3.3.1-Event-modifiers

- `.stop` → calls `event.stopPropagation()`
    
- `.prevent` → calls `event.preventDefault()`
    
- `.capture`, `.self`, `.once`, `.passive` for specific event behavior
    

#### 3.3.3.2-Model-modifiers

- `.lazy` → sync on change instead of input
    
- `.number` → cast input as number
    
- `.trim` → trim whitespace
    

#### 3.3.3.3-Binding-modifiers

- `.prop` → bind as a DOM property
    
- `.camel` → convert kebab-case to camelCase
    

---

## 3.4-Computed-Properties-And-Watchers

### 3.4.1-Computed-properties

- Cached based on reactive dependencies.
    
- Only re-evaluate when dependencies change.
    

### 3.4.2-Methods-vs-Computed

- Use computed when you need caching.
    
- Use methods for actions on demand.
    

### 3.4.3-Watchers

- Watchers allow executing code in response to data changes.
    
- Good for async tasks or complex watchers.
    

```js
watch: {
  question: function (newVal) {
    this.answer = 'Waiting...';
    this.fetchAnswer();
  }
}
```

---

## 3.5-Class-And-Style-Bindings

### 3.5.1-Object-syntax-for-class

```html
<div :class="{ active: isActive }"></div>
```

### 3.5.2-Array-syntax-for-class

```html
<div :class="[classA, classB]"></div>
```

### 3.5.3-Binding-inline-styles

```html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

### 3.5.4-Using-computed-properties-for-classes-styles

```js
computed: {
  classObject () {
    return {
      active: this.isActive,
      'text-danger': this.hasError
    }
  }
}
```

---

## 3.6-Conditional-Rendering

### 3.6.1-v-if-v-else-if-v-else

```html
<p v-if="seen">Now you see me</p>
<p v-else>Nope</p>
```

### 3.6.2-v-show

- Toggles visibility via CSS (`display: none`).
    
- Unlike `v-if`, it doesn’t remove/re-add element from DOM.
    

### 3.6.3-Key-differences

- `v-if`: Conditional block evaluation; heavy cost
    
- `v-show`: Toggle display; better for frequent toggles
    

### 3.6.4-template-tag-with-conditionals

```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph</p>
</template>
```

---

## 3.7-List-Rendering

### 3.7.1-v-for-with-arrays

```html
<li v-for="item in items">{{ item.text }}</li>
```

### 3.7.2-v-for-with-objects

```html
<div v-for="(value, key) in object">
  {{ key }}: {{ value }}
</div>
```

### 3.7.3-v-for-with-index

```html
<li v-for="(item, index) in items">{{ index }} - {{ item }}</li>
```

### 3.7.4-Key-binding

```html
<li v-for="item in items" :key="item.id">{{ item.text }}</li>
```

### 3.7.5-Nested-v-for

```html
<div v-for="group in groups">
  <div v-for="item in group.items">{{ item }}</div>
</div>
```

### 3.7.6-template-usage

- Use `<template>` with `v-for` when iterating over fragments.
    

---

## 3.8-Event-Handling

### 3.8.1-Listening-with-v-on

```html
<button v-on:click="handleClick">Click Me</button>
```

### 3.8.2-Method-handlers

- Reference a method in `methods` object.
    

### 3.8.3-Inline-handlers

```html
<button @click="count++">Add 1</button>
```

### 3.8.4-Event-object-access

```html
<button @click="handle($event)">Pass Event</button>
```

### 3.8.5-Event-modifiers

- `.stop`, `.prevent`, `.capture`, `.self`, `.once`, `.passive`
    

### 3.8.6-Key-modifiers

- Use `.enter`, `.esc`, `.tab`, etc.
    

```html
<input @keyup.enter="submit">
```

### 3.8.7-System-modifiers

- `.ctrl`, `.alt`, `.shift`, `.meta`
    

```html
<button @click.ctrl="save">Save</button>
```

### 3.8.8-Exact-modifier

- Ensures no other system modifiers are active.
    

```html
<button @click.exact="onClick">Exact</button>
```

---

## 3.9-Form-Input-Bindings

### 3.9.1-v-model-basics

- Two-way binding of input elements to data.
    

### 3.9.2-Input-types

```html
<input v-model="text">
<textarea v-model="message"></textarea>
<select v-model="selected">
  <option>A</option>
</select>
```

### 3.9.3-Multiple-checkboxes-select

```html
<input type="checkbox" v-model="checkedNames" value="Jack">
<select v-model="selectedOptions" multiple>
  <option>A</option>
</select>
```

### 3.9.4-Modifiers

- `.lazy`: Updates on change instead of input
    
- `.number`: Casts value to number
    
- `.trim`: Trims whitespace
    

---

## 3.10-Components-Basics

### 3.10.1-Global-and-local-registration

```js
Vue.component('my-component', { ... });
```

### 3.10.2-Props

- Data passed from parent to child
    
- Can validate using type, required, default
    

### 3.10.3-Template-syntax-for-props

```html
<child-comp :title="parentTitle"></child-comp>
```

### 3.10.4-Data-as-a-function

- Must return object in components to avoid shared state
    

### 3.10.5-Custom-events-with-$emit

```js
this.$emit('event-name', payload);
```

### 3.10.6-Parent-child-communication

- Props for down, `$emit` for up
    

### 3.10.7-One-root-element-rule

- Each component must return one root node in template
    

### 3.10.8-Reusability-and-encapsulation

- Components allow for modular, reusable, maintainable code