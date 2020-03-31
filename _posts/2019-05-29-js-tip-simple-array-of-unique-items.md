---
title:  "JS Tip: Simple Array Of Unique Items"
categories: 
- development 
- javascript
- ecmascript
---
A co-worker showed me a simple trick for getting an array of unique, simple values. Let's say we were combining two arrays of message id's:

```js
let arr = [19,22,7,12,6,85];
let arr2 = [22,8,3,19,45];
let newArr = [...arr, ...arr2];
// newArr equals [19, 22, 7, 12, 6, 85, 22, 8, 3, 19, 45]
```

This gives us a new array, combining the values of the first two. But, we often only want the unique values. Rather than looping over every item, checking for dupes, etc, we can take advantage of the new [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) object. A `Set` lets you store unique values of any type, and automatically tosses duplicates. As an iterable, it's easy to convert it from an Array like object to a true array.

```js
newArr = Array.from(new Set(newArr));
// newArr now equals [19, 22, 7, 12, 6, 85, 8, 3, 45]
```

And, being an array of numerics, we'd likely want to sort it in numeric order. We can do this with [Array.sort()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort).

```js
newArr = Array.from(new Set(newArr)).sort();
// Not exactly, now it reads [12, 19, 22, 3, 45, 6, 7, 8, 85]
```

OK, so that seems a little weird, until you read that documentation for `sort()` that I linked to above

<blockquote cite="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort">
The default sort order is built upon converting the elements into strings, then comparing their sequences of UTF-16 code units values.
</blockquote>

Well, that seems a bit of a bummer. But, you can get around this by using the optional `compareFunction` argument of the `sort()` method.

```js
newArr = Array.from(new Set(newArr)).sort((a,b) => a - b);
// That's better! Now it reads [3, 6, 7, 8, 12, 19, 22, 45, 85]
```

And there you have it. Simple, unique value array. The `Set` object allows for any type, so you <em>could</em> use complex objects as well, but again you would have to provide a custom `compareFunction` for handling the `sort()`.