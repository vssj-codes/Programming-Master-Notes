# Table-of-Contents

<!-- toc -->

- [5-Reusability-And-Composition](#5-reusability-and-composition)
  * [5.1-Mixins](#51-mixins)
    + [5.1.1-Defining-mixins](#511-defining-mixins)
    + [5.1.2-Using-mixins-in-components](#512-using-mixins-in-components)
    + [5.1.3-Merge-strategies](#513-merge-strategies)
    + [5.1.4-Global-vs-local-mixins](#514-global-vs-local-mixins)
    + [5.1.5-Naming-conflicts-and-overriding](#515-naming-conflicts-and-overriding)
  * [5.2-Custom-Directives](#52-custom-directives)
    + [5.2.1-Creating-directives](#521-creating-directives)
    + [5.2.2-Directive-hooks](#522-directive-hooks)
    + [5.2.3-Passing-values](#523-passing-values)
    + [5.2.4-Modifiers](#524-modifiers)
    + [5.2.5-Local-registration](#525-local-registration)
  * [5.3-Filters](#53-filters)
    + [5.3.1-What-filters-do](#531-what-filters-do)
    + [5.3.2-Global-filters](#532-global-filters)
    + [5.3.3-Local-filters](#533-local-filters)
    + [5.3.4-Chaining-filters](#534-chaining-filters)
    + [5.3.5-Limitations](#535-limitations)
  * [5.4-Plugins](#54-plugins)
    + [5.4.1-Creating-a-plugin](#541-creating-a-plugin)
    + [5.4.2-Global-assets](#542-global-assets)
    + [5.4.3-Injecting-options](#543-injecting-options)
    + [5.4.4-Using-plugins](#544-using-plugins)
    + [5.4.5-Best-practices](#545-best-practices)
  * [5.5-Render-Functions-And-JSX](#55-render-functions-and-jsx)
    + [5.5.1-Why-use-render-functions](#551-why-use-render-functions)
    + [5.5.2-createElement-arguments](#552-createelement-arguments)
    + [5.5.3-Functional-components](#553-functional-components)
    + [5.5.4-JSX-support](#554-jsx-support)
    + [5.5.5-Comparison-with-templates](#555-comparison-with-templates)
  * [5.6-Scoped-Slots](#56-scoped-slots)
    + [5.6.1-Basic-usage](#561-basic-usage)
    + [5.6.2-Slot-props](#562-slot-props)
    + [5.6.3-Parent-access](#563-parent-access)
    + [5.6.4-Dynamic-rendering](#564-dynamic-rendering)
    + [5.6.5-Caveats-and-best-practices](#565-caveats-and-best-practices)

<!-- tocstop -->

---

# 5-Reusability-And-Composition

## 5.1-Mixins

### 5.1.1-Defining-mixins

- A mixin is an object containing reusable options.
    

```js
const myMixin = {
  created() {
    console.log('Mixin hook called');
  },
  methods: {
    hello() {
      console.log('Hello from mixin');
    }
  }
}
```

### 5.1.2-Using-mixins-in-components

- Include in a component’s `mixins` array.
    

```js
mixins: [myMixin]
```

### 5.1.3-Merge-strategies

- Lifecycle hooks are merged.
    
- Methods and data are overwritten with component values if conflicts exist.
    

### 5.1.4-Global-vs-local-mixins

- Global mixins affect all components:
    

```js
Vue.mixin(myMixin)
```

- Use cautiously due to potential side effects.
    

### 5.1.5-Naming-conflicts-and-overriding

- Component methods/data take precedence over mixins if names conflict.
    

---

## 5.2-Custom-Directives

### 5.2.1-Creating-directives

```js
Vue.directive('focus', {
  inserted: function (el) {
    el.focus();
  }
});
```

### 5.2.2-Directive-hooks

- `bind`: once when bound.
    
- `inserted`: when inserted into parent.
    
- `update`: on update without children change.
    
- `componentUpdated`: after children updated.
    
- `unbind`: cleanup when directive removed.
    

### 5.2.3-Passing-values

```html
<input v-my-directive="someValue">
```

### 5.2.4-Modifiers

- Accessed via `binding.modifiers`
    

```html
<input v-my-directive.lazy.trim>
```

### 5.2.5-Local-registration

- Register in a component’s `directives` option.
    

---

## 5.3-Filters

### 5.3.1-What-filters-do

- Used to format data in templates.
    
- Synchronous, chainable string transformers.
    

### 5.3.2-Global-filters

```js
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  return value.charAt(0).toUpperCase() + value.slice(1)
})
```

### 5.3.3-Local-filters

```js
filters: {
  capitalize (value) {
    return value.toUpperCase()
  }
}
```

### 5.3.4-Chaining-filters

```html
{{ msg | capitalize | reverse }}
```

### 5.3.5-Limitations

- Filters cannot be used in JavaScript expressions.
    
- Deprecated in Vue 3.
    

---

## 5.4-Plugins

### 5.4.1-Creating-a-plugin

```js
const MyPlugin = {
  install(Vue, options) {
    Vue.prototype.$myMethod = function () { ... }
  }
}
```

### 5.4.2-Global-assets

- Add filters, directives, components globally.
    

```js
Vue.component('MyComp', MyComp);
```

### 5.4.3-Injecting-options

- Access options passed to `Vue.use()` in `install()`.
    

### 5.4.4-Using-plugins

```js
Vue.use(MyPlugin, { someOption: true });
```

### 5.4.5-Best-practices

- Always check if plugin has already been installed.
    
- Use namespacing to avoid conflicts.
    

---

## 5.5-Render-Functions-And-JSX

### 5.5.1-Why-use-render-functions

- Fine-grained control over rendering logic.
    
- Alternative to templates.
    

### 5.5.2-createElement-arguments

```js
render: function (createElement) {
  return createElement('h1', 'Hello')
}
```

- Params: tag, data object, children
    

### 5.5.3-Functional-components

- Set `functional: true`
    
- No reactive instance; stateless
    

```js
Vue.component('my-button', {
  functional: true,
  render(h, context) {
    return h('button', context.data, context.children);
  }
});
```

### 5.5.4-JSX-support

- Requires Babel plugin: `@vue/babel-preset-jsx`
    

```jsx
render () {
  return <h1>Hello JSX</h1>
}
```

### 5.5.5-Comparison-with-templates

- Templates are more declarative and readable.
    
- Render functions are more flexible.
    

---

## 5.6-Scoped-Slots

### 5.6.1-Basic-usage

```html
<child>
  <template v-slot:default="slotProps">
    {{ slotProps.text }}
  </template>
</child>
```

### 5.6.2-Slot-props

- Expose child data to parent scope using `v-bind` inside child.
    

### 5.6.3-Parent-access

- Access slot props using template `v-slot` directive.
    

### 5.6.4-Dynamic-rendering

- Can render dynamic lists or components based on slot props.
    

### 5.6.5-Caveats-and-best-practices

- Slots render in parent scope.
    
- Use descriptive names and avoid overuse for readability.