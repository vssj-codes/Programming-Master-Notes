# Table-of-Contents

<!-- toc -->

- [4-Components-In-Depth](#4-components-in-depth)
  * [4.1-Component-Registration](#41-component-registration)
    + [4.1.1-Global-registration](#411-global-registration)
    + [4.1.2-Local-registration](#412-local-registration)
    + [4.1.3-Naming-conventions](#413-naming-conventions)
    + [4.1.4-Recursive-components](#414-recursive-components)
    + [4.1.5-Circular-references](#415-circular-references)
  * [4.2-Props](#42-props)
    + [4.2.1-Passing-props](#421-passing-props)
    + [4.2.2-Static-vs-dynamic-props](#422-static-vs-dynamic-props)
    + [4.2.3-Prop-validation](#423-prop-validation)
      - [4.2.3.1-Type-check](#4231-type-check)
      - [4.2.3.2-Required](#4232-required)
      - [4.2.3.3-Default-values](#4233-default-values)
      - [4.2.3.4-Custom-validator-functions](#4234-custom-validator-functions)
    + [4.2.4-One-way-data-flow](#424-one-way-data-flow)
    + [4.2.5-Sync-modifier-and-update-event](#425-sync-modifier-and-update-event)
    + [4.2.6-inheritAttrs-false](#426-inheritattrs-false)
  * [4.3-Custom-Events](#43-custom-events)
    + [4.3.1-Emitting-events](#431-emitting-events)
    + [4.3.2-Listening-with-v-on](#432-listening-with-v-on)
    + [4.3.3-Event-names-and-casing](#433-event-names-and-casing)
    + [4.3.4-Using-$listeners](#434-using-listeners)
    + [4.3.5-Using-$attrs](#435-using-attrs)
    + [4.3.6-Custom-v-model-binding](#436-custom-v-model-binding)
  * [4.4-Slots](#44-slots)
    + [4.4.1-Default-slot](#441-default-slot)
    + [4.4.2-Named-slots](#442-named-slots)
    + [4.4.3-Scoped-slots](#443-scoped-slots)
    + [4.4.4-Fallback-content](#444-fallback-content)
    + [4.4.5-Slot-shorthand](#445-slot-shorthand)
    + [4.4.6-Slot-rendering-caveats](#446-slot-rendering-caveats)
  * [4.5-Dynamic-Components](#45-dynamic-components)
    + [4.5.1-component-is-syntax](#451-component-is-syntax)
    + [4.5.2-Preserving-state-with-key](#452-preserving-state-with-key)
    + [4.5.3-keep-alive](#453-keep-alive)
    + [4.5.4-Async-components-with-import](#454-async-components-with-import)
  * [4.6-Async-Components](#46-async-components)
    + [4.6.1-Factory-function](#461-factory-function)
    + [4.6.2-Webpack-code-splitting](#462-webpack-code-splitting)
    + [4.6.3-Use-with-v-if-or-v-show](#463-use-with-v-if-or-v-show)
    + [4.6.4-Error-handling-loading-state](#464-error-handling-loading-state)
  * [4.7-Advanced-Component-Patterns](#47-advanced-component-patterns)
    + [4.7.1-Provide-and-inject](#471-provide-and-inject)
    + [4.7.2-Mixins](#472-mixins)
    + [4.7.3-Extending-components](#473-extending-components)
    + [4.7.4-Functional-components](#474-functional-components)
    + [4.7.5-\$parent-and-$root](#475-parent-and-root)
    + [4.7.6-Recursive-components](#476-recursive-components)
    + [4.7.7-Dynamic-template-generation-caveats](#477-dynamic-template-generation-caveats)

<!-- tocstop -->

---

# 4-Components-In-Depth

## 4.1-Component-Registration

### 4.1.1-Global-registration

- Use `Vue.component('my-comp', {...})` to register a component globally.
    
- Available in all descendant components.
    

### 4.1.2-Local-registration

- Register inside `components` option of a Vue instance/component.
    

```js
components: {
  'my-comp': MyComp
}
```

### 4.1.3-Naming-conventions

- Use `PascalCase` in JavaScript, `kebab-case` in templates.
    
- Case-sensitive on native HTML but case-insensitive in DOM.
    

### 4.1.4-Recursive-components

- A component can reference itself if it includes a `name` option.
    

```js
name: 'TreeItem'
```

### 4.1.5-Circular-references

- Use the `name` option and asynchronous `component` registration to resolve circular references.
    

---

## 4.2-Props

### 4.2.1-Passing-props

```html
<child :title="parentTitle"></child>
```

### 4.2.2-Static-vs-dynamic-props

- Static: `<comp title="Static">`
    
- Dynamic: `:title="dynamicTitle"`
    

### 4.2.3-Prop-validation

- Use `props` as an object to specify rules.
    

```js
props: {
  title: String,
  likes: {
    type: Number,
    required: true,
    default: 0,
    validator: value => value >= 0
  }
}
```

#### 4.2.3.1-Type-check

- Validate against constructor (String, Number, etc.)
    

#### 4.2.3.2-Required

- Use `required: true` for mandatory props.
    

#### 4.2.3.3-Default-values

- Use `default` key; if object or array, must be a factory function.
    

#### 4.2.3.4-Custom-validator-functions

- Provide custom logic to accept or reject prop values.
    

### 4.2.4-One-way-data-flow

- Parent → Child only.
    
- Never mutate a prop inside a child.
    

### 4.2.5-Sync-modifier-and-update-event

```html
<child :title.sync="parentTitle"></child>
```

- Equivalent to listening for `update:title` event.
    

### 4.2.6-inheritAttrs-false

- Prevent automatic attribute inheritance on root element.
    
- Useful in form input wrappers.
    

---

## 4.3-Custom-Events

### 4.3.1-Emitting-events

```js
this.$emit('event-name', payload);
```

### 4.3.2-Listening-with-v-on

```html
<child @custom-event="handleEvent"></child>
```

### 4.3.3-Event-names-and-casing

- DOM attributes are case-insensitive.
    
- Use kebab-case in templates and camelCase in JS.
    

### 4.3.4-Using-$listeners

- Bind all parent listeners using `$listeners`.
    

```html
<input v-bind="$listeners">
```

### 4.3.5-Using-$attrs

- Inherit non-prop attributes to child components.
    
- Often combined with `inheritAttrs: false` and `$attrs`.
    

### 4.3.6-Custom-v-model-binding

- Customize `v-model` with `model` option:
    

```js
model: {
  prop: 'checked',
  event: 'change'
}
```

---

## 4.4-Slots

### 4.4.1-Default-slot

```html
<slot></slot>
```

### 4.4.2-Named-slots

```html
<slot name="header"></slot>
```

```html
<template v-slot:header>Header content</template>
```

### 4.4.3-Scoped-slots

- Expose data from child to slot using a scoped slot.
    

```html
<slot :text="text"></slot>
```

```html
<template v-slot:default="slotProps">
  {{ slotProps.text }}
</template>
```

### 4.4.4-Fallback-content

- Provide default content when no slot content is passed.
    

### 4.4.5-Slot-shorthand

- `v-slot:name` can be written as `#name`
    

### 4.4.6-Slot-rendering-caveats

- Each slot is rendered only once per scope.
    

---

## 4.5-Dynamic-Components

### 4.5.1-component-is-syntax

```html
<component :is="currentComponent"></component>
```

### 4.5.2-Preserving-state-with-key

- Use `:key` to control re-rendering and preserve state.
    

### 4.5.3-keep-alive

- Cache component state for toggled components.
    

```html
<keep-alive>
  <component :is="view"></component>
</keep-alive>
```

### 4.5.4-Async-components-with-import

```js
const AsyncComp = () => import('./MyComp.vue');
```

---

## 4.6-Async-Components

### 4.6.1-Factory-function

```js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(() => {
    resolve({ template: '<div>I am async!</div>' });
  }, 1000);
});
```

### 4.6.2-Webpack-code-splitting

```js
const AsyncComp = () => import('./MyComp.vue');
```

### 4.6.3-Use-with-v-if-or-v-show

- Control conditional display of async components.
    

### 4.6.4-Error-handling-loading-state

```js
Vue.component('async-comp', () => ({
  component: import('./Comp.vue'),
  loading: LoadingComponent,
  error: ErrorComponent,
  delay: 200,
  timeout: 3000
}))
```

---

## 4.7-Advanced-Component-Patterns

### 4.7.1-Provide-and-inject

- Dependency injection for deeply nested components.
    

```js
provide: { theme: 'dark' }
inject: ['theme']
```

### 4.7.2-Mixins

- Reuse options/data/methods.
    
- Merged into consuming components.
    

### 4.7.3-Extending-components

```js
const CompA = Vue.extend({ ... });
```

### 4.7.4-Functional-components

- Stateless, instanceless.
    

```js
functional: true
```

### 4.7.5-\$parent-and-$root

- `$parent`: immediate parent instance
    
- `$root`: root Vue instance
    

### 4.7.6-Recursive-components

- A component that renders itself.
    
- Useful for tree structures.
    

### 4.7.7-Dynamic-template-generation-caveats

- Templates must have one root node
    
- Cannot define templates dynamically at runtime