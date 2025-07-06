# Table-of-Contents

<!-- toc -->

- [7.1-Installation-And-Setup](#71-installation-and-setup)
  * [7.1.1-Installing-Vue-Router](#711-installing-vue-router)
  * [7.1.2-Creating-the-router-instance](#712-creating-the-router-instance)
  * [7.1.3-Integrating-router-with-root-instance](#713-integrating-router-with-root-instance)
  * [7.1.4-router-view-and-router-link](#714-router-view-and-router-link)
  * [7.1.5-Basic-example](#715-basic-example)
- [7.2-Routes-Configuration](#72-routes-configuration)
  * [7.2.1-Static-routes](#721-static-routes)
  * [7.2.2-Dynamic-route-matching](#722-dynamic-route-matching)
  * [7.2.3-Nested-routes](#723-nested-routes)
  * [7.2.4-Named-routes](#724-named-routes)
  * [7.2.5-Redirects-and-aliases](#725-redirects-and-aliases)
  * [7.2.6-Catch-all-route](#726-catch-all-route)
- [7.3-Navigation](#73-navigation)
  * [7.3.1-router-link-component](#731-router-link-component)
  * [7.3.2-Programmatic-navigation](#732-programmatic-navigation)
  * [7.3.3-Relative-and-absolute-paths](#733-relative-and-absolute-paths)
  * [7.3.4-Navigation-with-params-query](#734-navigation-with-params-query)
  * [7.3.5-Replace-navigation-history](#735-replace-navigation-history)
  * [7.3.6-Active-class-styling](#736-active-class-styling)
- [7.4-Navigation-Guards](#74-navigation-guards)
  * [7.4.1-Global-guards](#741-global-guards)
  * [7.4.2-Per-route-guards](#742-per-route-guards)
  * [7.4.3-In-component-guards](#743-in-component-guards)
  * [7.4.4-Asynchronous-navigation](#744-asynchronous-navigation)
  * [7.4.5-next-options](#745-next-options)
- [7.5-Route-Props](#75-route-props)
  * [7.5.1-Boolean-mode](#751-boolean-mode)
  * [7.5.2-Object-mode](#752-object-mode)
  * [7.5.3-Function-mode](#753-function-mode)
- [7.6-Lazy-Loading-Routes](#76-lazy-loading-routes)
  * [7.6.1-Code-splitting](#761-code-splitting)
  * [7.6.2-Webpack-usage](#762-webpack-usage)
  * [7.6.3-webpackChunkName](#763-webpackchunkname)
- [7.7-Scroll-Behavior](#77-scroll-behavior)
  * [7.7.1-Enabling-scroll-position](#771-enabling-scroll-position)
  * [7.7.2-scrollBehavior-function](#772-scrollbehavior-function)
  * [7.7.3-Manual-scroll-restore](#773-manual-scroll-restore)
- [7.8-Named-Views](#78-named-views)
  * [7.8.1-Multiple-router-views](#781-multiple-router-views)
  * [7.8.2-Route-components-object](#782-route-components-object)
  * [7.8.3-Use-case](#783-use-case)
- [7.9-Redirects-And-Aliases](#79-redirects-and-aliases)
  * [7.9.1-Redirects](#791-redirects)
  * [7.9.2-Aliases](#792-aliases)
  * [7.9.3-Differences](#793-differences)
- [7.10-Wildcard-And-Catch-All-Routes](#710-wildcard-and-catch-all-routes)
  * [7.10.1-Wildcard-usage](#7101-wildcard-usage)
  * [7.10.2-404-page](#7102-404-page)
  * [7.10.3-Dynamic-fallback-logic](#7103-dynamic-fallback-logic)
- [7.11-Advanced-Features](#711-advanced-features)
  * [7.11.1-Navigation-failures](#7111-navigation-failures)
  * [7.11.2-Route-meta-fields](#7112-route-meta-fields)
  * [7.11.3-Scroll-animations](#7113-scroll-animations)
  * [7.11.4-History-vs-hash-mode](#7114-history-vs-hash-mode)
  * [7.11.5-addRoutes-dynamic-matching](#7115-addroutes-dynamic-matching)

<!-- tocstop -->

---

## 7.1-Installation-And-Setup

### 7.1.1-Installing-Vue-Router

- Install using npm:
    

```bash
npm install vue-router
```

### 7.1.2-Creating-the-router-instance

```js
import Vue from 'vue';
import VueRouter from 'vue-router';
import Home from './views/Home.vue';

Vue.use(VueRouter);

const routes = [
  { path: '/', component: Home }
];

const router = new VueRouter({
  routes
});
```

### 7.1.3-Integrating-router-with-root-instance

```js
new Vue({
  router,
  render: h => h(App)
}).$mount('#app');
```

### 7.1.4-router-view-and-router-link

- `<router-view>`: Placeholder for matched component
    
- `<router-link>`: Navigation component
    

```html
<router-link to="/">Home</router-link>
```

### 7.1.5-Basic-example

```js
const routes = [
  { path: '/about', component: About }
];
```

---

## 7.2-Routes-Configuration

### 7.2.1-Static-routes

- Map paths to components statically.
    

### 7.2.2-Dynamic-route-matching

```js
{ path: '/user/:id', component: User }
```

- Access with `this.$route.params.id`
    

### 7.2.3-Nested-routes

```js
{
  path: '/parent',
  component: Parent,
  children: [
    { path: 'child', component: Child }
  ]
}
```

### 7.2.4-Named-routes

```js
{ path: '/user/:id', name: 'user', component: User }
```

### 7.2.5-Redirects-and-aliases

```js
{ path: '/a', redirect: '/b' }
{ path: '/c', alias: '/d' }
```

### 7.2.6-Catch-all-route

```js
{ path: '*', component: NotFound }
```

---

## 7.3-Navigation

### 7.3.1-router-link-component

```html
<router-link to="/home">Home</router-link>
```

### 7.3.2-Programmatic-navigation

```js
this.$router.push('/home');
```

### 7.3.3-Relative-and-absolute-paths

- Absolute: `/profile`
    
- Relative: `profile` (relative to current path)
    

### 7.3.4-Navigation-with-params-query

```js
this.$router.push({ name: 'user', params: { id: 123 } });
```

### 7.3.5-Replace-navigation-history

```js
this.$router.replace('/login');
```

### 7.3.6-Active-class-styling

- `router-link-active`, `router-link-exact-active`
    
- Customizable via props `active-class`, `exact-active-class`
    

---

## 7.4-Navigation-Guards

### 7.4.1-Global-guards

```js
router.beforeEach((to, from, next) => {
  next();
});
```

### 7.4.2-Per-route-guards

```js
{
  path: '/admin',
  component: Admin,
  beforeEnter: (to, from, next) => {
    next();
  }
}
```

### 7.4.3-In-component-guards

```js
beforeRouteEnter (to, from, next) {}
beforeRouteUpdate (to, from, next) {}
beforeRouteLeave (to, from, next) {}
```

### 7.4.4-Asynchronous-navigation

- Use `next()` after async operations complete.
    

### 7.4.5-next-options

- `next()` → proceed
    
- `next(false)` → cancel
    
- `next({ path: '/login' })` → redirect
    

---

## 7.5-Route-Props

### 7.5.1-Boolean-mode

- Automatically passes `route.params` as props.
    

```js
{ path: '/user/:id', component: User, props: true }
```

### 7.5.2-Object-mode

```js
props: { id: 123 }
```

### 7.5.3-Function-mode

```js
props: route => ({ id: route.params.id })
```

---

## 7.6-Lazy-Loading-Routes

### 7.6.1-Code-splitting

```js
const Foo = () => import('./Foo.vue');
```

### 7.6.2-Webpack-usage

- Enables route-based chunking
    

### 7.6.3-webpackChunkName

```js
const Bar = () => import(/* webpackChunkName: "group-bar" */ './Bar.vue');
```

---

## 7.7-Scroll-Behavior

### 7.7.1-Enabling-scroll-position

```js
const router = new VueRouter({
  scrollBehavior (to, from, savedPosition) {
    return { x: 0, y: 0 };
  }
});
```

### 7.7.2-scrollBehavior-function

- Control scroll position when navigating.
    

### 7.7.3-Manual-scroll-restore

- Use browser APIs for fine control.
    

---

## 7.8-Named-Views

### 7.8.1-Multiple-router-views

```html
<router-view name="default" />
<router-view name="sidebar" />
```

### 7.8.2-Route-components-object

```js
components: {
  default: MainView,
  sidebar: SidebarView
}
```

### 7.8.3-Use-case

- Layouts with multiple sections.
    

---

## 7.9-Redirects-And-Aliases

### 7.9.1-Redirects

```js
{ path: '/home', redirect: '/' }
```

### 7.9.2-Aliases

```js
{ path: '/official', alias: '/gov' }
```

### 7.9.3-Differences

- Redirect changes the path.
    
- Alias doesn’t update the URL.
    

---

## 7.10-Wildcard-And-Catch-All-Routes

### 7.10.1-Wildcard-usage

```js
{ path: '*', component: NotFound }
```

### 7.10.2-404-page

- Catch unmatched routes and show fallback.
    

### 7.10.3-Dynamic-fallback-logic

- You can redirect or conditionally show error pages.
    

---

## 7.11-Advanced-Features

### 7.11.1-Navigation-failures

```js
this.$router.push('/path').catch(err => {});
```

### 7.11.2-Route-meta-fields

```js
meta: { requiresAuth: true }
```

- Access via `to.meta`
    

### 7.11.3-Scroll-animations

- Combine scroll behavior with transitions.
    

### 7.11.4-History-vs-hash-mode

- Hash mode: `/#/about`
    
- History mode: clean URLs, needs server config
    

### 7.11.5-addRoutes-dynamic-matching

```js
router.addRoutes([ ... ])
```

- Add routes at runtime (Vue Router 3 only)
    

---