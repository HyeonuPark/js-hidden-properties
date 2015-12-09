# Summary

Transform underscore prefixed properties to symbol.

In JavaScript, de faqto way to make object's private property is prefix that property's name with _ (underscore), because JavaScript doesn't provide any access control of property, in natural way.

In this way, there's a chance of name collision when multiple modules share a object. Underscore prefixed properties are generally not in that module's public api, so other modules may not know that property is used.

In ES2015, there's a Symbol, unique identifier of property regardless of it's name. But using symbol directly is not really natural then traditional way. We need another spoon of sugar.

# Detailed Design

Each module of JavaScript code has it's own Symbol pool.

Underscore prefixed keyword is treated as treated as a Symbol from module's pool.

Explicitly declared underscore prefixed keywords will be ignored.

Symbols can be import/export from/to another module.

# Desugaring

## Object Property

```js
let obj = {}
obj._hiddenProp = 'foo'
console.log(obj._hiddenProp)
```

is equal to

```js
const _hiddenProp = Symbol('hiddenProp')

let obj = {}
obj[_hiddenProp] = 'foo'
console.log(obj[_hiddenProp])
```

## Object Expression

```js
let obj = {
  foo: 'bar',
  _hiddenFoo: 'hiddenBar'
  _getFoo() {
    return this.foo
  }
}
```

is equal to

```js
const _hiddenFoo = Symbol('hiddenFoo')
const _getFoo = Symbol('getFoo')

let obj = {
  foo: 'bar',
  [_hiddenFoo]: 'hiddenBar',
  [_getFoo]() {
    return this.foo
  }
}
```

## Class

```js
class MyClass {
  method () {
    console.log('public')
  }

  _private () {
    console.log('private')
  }
}
```

is equal to

```js
const _private = Symbol('private')

class MyClass {
  method () {
    console.log('public')
  }

  [_private] () {
    console.log('private')
  }
}
```

## Import/Export

```js
import {obj, _hidden as _origin} from 'anotherModule'

obj._hidden = 'foo'
console.log(obj._origin)

export {obj, _hidden}
```

is equal to

```js
import {obj, _hidden as _origin} from 'anotehrModule'

const _hidden = Symbol('hidden')

obj[_hidden] = 'foo'
console.log(obj[_origin])

export {obj, _hidden}
```

## Closure

```js
console.log((function () {
  return {
    _hidden: 'foo'
  }
})())
```

is equal to

```js
const _hidden = Symbol('hidden')

console.log((function () {
  return {
    [_hidden]: 'foo'
  }
})())
```

## Explicit Declaration

```js
let _hidden = 'NotASymbol'

let obj = {
  _hidden: 'foo'
}
```

is equal to

```js
let _hidden = 'NotASymbol'

let obj = {
  [_hidden]: 'foo'
}
```
