# This is my essay

This function adds stuff:

```js
// index.js
export default function add (a, b) {
  return a + b
}
```

And also write a test for it:

```js
// index.test.js
import add from './'
it('should add two numbers', () => {
  assert(add(1, 2) === 3)
})
```
