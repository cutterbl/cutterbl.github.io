---
title:  "Death to Var - Why Let and Const Really Interest Me In JavaScript"
categories: 
- development 
- javascript
- ecmascript
---
Today I want to talk about the value of ES2015's new *let* and *const* variable declarations, and
give you some use case scenarios. But first, let me tell you why I was really looking at all.

[Ben Nadel](http://bennadel.com) is one of my favorite people. You will not, ever, 
meet a nicer guy. Ben is the kind of guy that if the two of you were walking down the street in a blizzard, and you
were cold, he'd give you the shirt off of his own back and go topless so you wouldn't freeze. Yes, he really is
that nice of a guy.

I'd like to say that I've learned many things from Ben over the years. He blogs about everything as he learns
it, sharing what he finds along the way. And he's the first to tell you that he's not always right. Sometimes the
comments to his posts are even more informative than the posts themselves. And, sometimes, he gives his opinion
on a matter of programming and that opinion might not always follow best practice.

About a week ago, Ben posted an article titled [Var For Life - Why Let And Const Don't Interest Me In JavaScript](http://www.bennadel.com/blog/2949-var-for-life---why-let-and-const-don-t-interest-me-in-javascript.htm). He's very clear, in his post,
saying that his article is an **opinion piece**. His thoughts are clear, his examples make sense, and
it's easy to see where he's coming from. You'll also find some really thought provoking discussion in the comment
thread both for and against.

But I think it's important to truly explore these new constructs in JavaScript. They were introduced with one
true goal in mind: to help manage memory in our applications. With the proliferation of JavaScript based applications,
both client-side and server-side, the need to carefully analyze our architecture has increased a dozen fold. How
you manage your variable declarations will directly impact your overall memory utilization, as well as assist you
in preventing race conditions within your app. The *let* and *const* declarations really fine tune
that control.

The *let* declaration construct is fairly straightforward. It is a block level scoping mechanism,
supplanting *var* usage in most situations, and controls the "this" level access of those variables. 
The *var* declaration construct was a function level scope. What's the difference between block level
scoping and function level scoping? Consider the following:

```js
	for (var i = 0; i < 10; i++) {
		console.log('i = ', i);
	}
	console.log('now we are outside of our block. i = ', i); // i now equals 10
```

Function level scoping means that variables declared using the *var* construct are available only
within the confines of that function, but are not restricted to the block they are declared within. Running
the above example shows you that **i** still exists outside of the for loop block. What happens
though if we change that declaration to a block level declaration?

```js
	for (let i = 0; i < 10; i++) {
		console.log('i = ', i);
	}
	console.log('now we are outside of our block. i = ', i); // throws an error that i doesn't exist
```

In the case above, the variable **i** is now a block scoped variable and, as such, is **only**
available within the confines of the for loop. The variable is cleared from memory once execution is complete
(since there are no references created to those variables in the block), and their values are not available
outside of the block, reducing the opportunity for race conditions.

Probably the most misunderstood of these constructs is the *const* form of variable declaration. Most
still think of this as setting an immutable **constant**, but that's not entirely correct. Let me give
you an example:

```js
	const myVar = 'JavaScript is really ECMAScript';
	console.log(myVar.replace('really ', ''));
	myVar = 'Purple Haze'; // This throws an error, because you can't do this
```

OK, that example supports that whole "immutable constant" kinda thing. But that isn't the whole story. Let's
look at another example:

```js
	const myVar = {};
	myVar.foo = 'bar';
	console.log(myVar.foo);
	myVar = {}; // You were just fine til you got to this line
```

"Wait? What?" Yes, you can change a variable declared with *const*. Sorta.

When you set a variable with *const*, you are assigning a variable to a specific location in memory.
It is set to the type you initially assign. You can adjust properties of that variable, but you can not replace
the variable, even with one of the same type. This is why examples with a simple type (string or numeric or
boolean) would throw an error, but you could create and remove and adjust object keys or array elements all day
long. The variable itself isn't *constant*, it's location in memory is.

Which allows me to change an example from a [previous post](http://www.cutterscrossing.com/index.cfm/2015/11/16/Angular-Data-and-ES2015-Classes). In that post, I talked about using implicit ES2015 getters and setters, and showed an example
of broadcasting a variable change in a service from within a custom setter method. I had a variable in my
Controller that was not passed in to the Service by reference, so any time I changed the Service variable it had
to broadcast that change to my Controller so I could update the controller level variable. In my original example, 
the variable was originally assigned to the class' "this" scope. But with *const* I can assign that variable 
and hold it's location in memory, thereby passing the memory reference and changing how I can control workflow.

```js
	'use strict';
	
	class MyController {
		constructor ($scope, dataService, orderService) {
			this.$scope = $scope;
			this._dataSvc = dataService;
			this._orderSvc = orderService;
			
			const myCrazyVar = {};
			// setting to 'this' too, for controller public accessable reference
			dataService.myCrazyVar = this.myCrazyVar = myCrazyVar;
		}
	}
	
	myController.$inject = ['$scope', 'dataService', 'orderService'];
	
	export {MyController};
```

```js
	'use strict';
	
	class DataService {
		constructor () {
			this.myCrazyVar = null;
		}
	}
	
	export {DataService};
```

```js
	'use strict';
	
	class OrderService {
		constructor (dataService) {
			this._dataSvc = dataService;
		}
		
		add (order) {
			// update our shared data
			this._dataSvc.myCrazyVar.orderid = order.id;
		}
	}
	
	orderService.$inject = ['dataService'];
	
	export {OrderService};
```

Is this wise? I'm sure if you aren't careful you can create issues. But, by passing that memory reference
around you also eliminate the need to duplicate variables and broadcast events unnecessarily, reducing your memory
footprint and cpu utilization.

Learning when to use *let* and when to use *const* will take some time for many who've worked
with JavaScript for any length of time. I'm sure this will be one of those new features that takes some significant
time to gain true traction among developers. In the end run, it will force us all to think ahead about the
architecture of our applications in advance (always a good thing), and the impact of our code on performance.

Now, if I can just convince Ben ;)