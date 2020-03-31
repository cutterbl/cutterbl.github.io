---
title:  "ES2015 and Fun With Parameters"
categories: 
- development 
- javascript
- ecmascript
- es2015
- angularjs
---

If you've come to JavaScript after learning to program in other languages, one thing that's probably stuck in
your craw over the years has been the lack of any way to define default parameters in functions. You've probably
written something like this in the past:

```js
	var foo = function (bar) {
		bar = bar || 'ok';
		// ...
	};
```

For the most part that sorta thing probably worked out, until your argument was a boolean, which then really
complicated things.

With ES2015, the heavens have opened and prayers have been answered, as we've finally been given the ability to
define default parameters. Consider the following:

```js
	// This is in a class
	foo (bar=true) {
		if (bar) {
			console.log('First round is on Simon!');
		} else {
			console.log('No drinks today!');
		}
	}
```

Simple. We're saying that if *bar* is *true*, then Simon is buying, otherwise we're out of luck,
and defaulting our argument to true. We can then call this method a few times to test it out:

```js
	constructor () {
		this.foo(); // 'First round is on Simon!'
		this.foo(false); // 'No drinks today'
		let bar;
		this.foo(bar); // 'First round is on Simon!'
	}
```

You can see from my comments what the output of those methods would be. It's important to note here that even
the *undefined* value passed as an argument triggered the default argument value as well.

Hopefully default arguments will help you to significantly simplify your code in the future.