# hyperviews

View template language that targets hyperscript.

Turns this template

```html
<div>
  <h1>{state.title} by {state.author}</h1>
  <p>Name: {state.name}</p>
  <button type=button onclick={actions.click()}></button>
  <if condition="state.isLoggedIn">
    <a href="/logout">Logout</a>
    <ul>
      <li each="post in state.posts" key={post.slug}>
        <a href="/post/post.id" title={post.title}>{post.name}</a>
      </li>
    </ul>
  <else>
    <a href="/login">Login</a>
  </if>
</div>
```

into this JavaScript

```js
h('div', {}, [
  h('h1', {}, (state.title) + ' by ' + (state.author)),
  h('p', {}, 'Name: ' + (state.name)),
  h('button', { type: 'button', onclick: function (e) { actions.click() } }),
  (function () {
    if (state.isLoggedIn) {
      return [
        h('a', { href: '/logout' }, 'Logout'),
        h('ul', {}, (state.posts || []).map(function ($value, $index, $target) {
          const post = $value
          return h('li', { key: (post.slug) }, h('a', { href: '/post/post.id', title: (post.title) }, (post.name)))
        }))
      ]
    } else {
      return h('a', { href: '/login' }, 'Login')
    }
  })()
])
```



## Installation

`npm i hyperviews`



## API

`hyperviews(tmpl, mode, name, argstr)`

- `tmpl` (required) - The template string.  
- `mode` - The output format. Can be one of [`raw`, `esm`, `cjs`, `browser`], if any other value is passed the function is exported as a variable with that name. The default is `raw`.
- `name` - The output function name (can be overridden with a `<template>` element). The default is `view`.
- `args` - The output function arguments (can be overridden with a `<template>` element). The default is `state actions`.



## CLI

Reads the template from stdin, 

`cat examples/test.html | hyperviews --mode esm --name foo --args bar > examples/test.js`

See [more CLI examples](./test/interpolation.js)


## Basics
```js
const hv = require('hyperviews')

hv("<div id='foo'>{state.name}</div>")
// => h('div', { id: 'foo' }, (state.name))
```



## Interpolation

Use curly braces in attributes and text.

```html
<div>
  <a class={state.class} href="http://www.google.co.uk?q={state.query}"></a>
  My name is {state.name} my age is {state.age} and I live at {state.address}
</div>
```

See [more interpolation examples](./test/interpolation.js)



## Conditionals

There are two forms of conditional.

Using an `if` attribute.

```html
<span if="state.bar === 1">Show Me!</span>
```

Or using tags `<if>`, `<elseif>` and `<else>`

```html
<div>
  <if condition="state.bar === 1">
    <span>1</span>
  <elseif condition="state.bar === 2">
    <span>2</span>
  <else>
    <span>bar is neither 1 or 2, it's {state.bar}!</span>
  </if>
</div>
```

`if` tags can be [nested](./test/conditionals.js#L84).

See [more conditional examples](./test/conditionals.js)



## Iteration

The `each` attribute can be used to repeat over items in an Array.
Three additional variables are available during each iteration: `$value`, `$index` and `$target`.

It supports keyed elements as shown here.

```html
<ul>
  <li each="post in state.posts" key={post.id}>
    <span>{post.title} {$index}</span>
  </li>
</ul>
```

produces

```js
h('ul', {}, (state.posts || []).map(function ($value, $index, $target) {
  const post = $value
  return h('li', { key: (post.id) }, h('span', {}, (post.title) + ' ' + ($index)))
}))
```

See [more iteration examples](./test/iteration.js)

## Events

```html
<a href="http://example.com" onclick={actions.do()}>{state.foo}</a>
```

produces this output


```js
h('a', { href: 'http://example.com', onclick: function (e) { actions.do() } }, (state.foo))
```

See [more event examples](./test/events.js)

## Style

The `style` attribute expects an object

```html
<p style="{ color: '#ddd', fontSize: '12px' }"></p>
```

produces this output

```js
h('p', { style: { color: '#ddd', fontSize: '12px' } })
```



Based on superviews.js, a [similar project](https://github.com/davidjamesstone/superviews.js) for [IncrementalDOM](https://github.com/google/incremental-dom) 



# Example

Todo list

```html
<section>
  <!-- Add new todo and toggle -->
  <form onsubmit={actions.add(e)}>
    <input type=text name=text class=form-control
      placeholder="Enter new todo" value={state.input}
      required=required autocomplete=off
      onchange="{state.input = this.value.trim()}">
    <input type=checkbox onchange={actions.toggleAll(this.checked)}>
  </form>

  <!-- Todos list -->
  <ul>
    <li each="todo in state.todos" key={todo.id}>
      {$index + 1}.
      <input type=text value={todo.text}
        onchange="{actions.updateText({id: todo.id, text: this.value})}" class=form-control
        style="{ borderColor: todo.text ? '': 'red', textDecoration: todo.done ? 'line-through': '' }">
      <input type=checkbox checked={todo.done}
        onchange={actions.toggleDone(todo.id)}>
    </li>
  </ul>

  <!-- Summary -->
  <span>Total {state.todos.length}</span>
  <button if="state.todos.find(t => t.done)" 
    onclick={actions.clearCompleted()}>
    Clear completed
  </button>

  <!-- Debug -->
  <pre><code>{JSON.stringify(state, null, 2)}</code></pre>
</section>
```

```js
  const { h, app } = hyperapp

  const state = {
    input: '',
    todos: [{
      id: 1,
      text: 'Buy milk'
    }, {
      id: 2,
      text: 'Buy coffee'
    }, {
      id: 3,
      text: 'Buy sugar',
      done: true
    }]
  }

  const actions = {
    add: (e) => state => {
      e.preventDefault()
      return {
        input: '',
        todos: state.todos.concat({
          id: Date.now(),
          text: state.input
        })
      }
    },
    updateText: (todo) => state => ({
      todos: state.todos.map(item => {
        if (item.id === todo.id) {
          item.text = todo.text
        }
        return item
      })
    }),
    toggleDone: (id) => state => ({
      todos: state.todos.map(item => {
        if (item.id === id) {
          item.done = !item.done
        }
        return item
      })
    }),
    clearCompleted: () => state => ({
      todos: state.todos.filter(t => !t.done)
    }),
    toggleAll: (value) => state => ({
      todos: state.todos.map(item => {
        item.done = value
        return item
      })
    })
  }

  app(state, actions, view, document.body)
```

Download the repo and open [the todo example](./examples/todo.html) to see it work.