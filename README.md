## The keyword `this` ##
### Objectives ###
- Define what the keyword 'this' is
- Understand the four ways to always figure out what the keyword 'this' is
- Try as hard as possible not to use the word "this" in a sentence

<hr>

### So what is `this`? ###
- A reserved keyword in JavaScript
- Usually determined by how a function is called (what we call 'execution context')
- Can be determined using four rules:
	1. [global](#1-global-context)
	2. [object/implicit](#2-object-implicit)
	3. [explicit](#3-explicit-binding)
        1. [`call()`](#-call-)
        2. [`apply()`](#-apply-)
        3. [`bind()`](#-bind-)
	4. [new](#4-new)

<hr>

### 1. Global Context
When `this` is not inside of a declared object, it refers to the `Window` object.
```javascript
console.log(this) // window
```
Therefore, when a variable is attached to `this`, we are actually just creating a global variable
```javascript
this.person1 = "Steve";
var person2 = "Chris";
person3 = "Dwayne";
console-log(person1); // "Steve"
console-log(person2); // "Chris"
console-log(person3); // "Dwayne"
```
A variable's scope *should* be limited to within the function that it was declared in.
The problem with using `this` is that we can accidently create global variables within a function.
```javascript
function badCodeExample(){
    this.food1 = "Cheeseburger";
    var food2 = "Hotdog";
    food3 = "Pizza";
}
badCodeExample();
console.log(food1); // "Cheeseburger"
console.log(food2); // ReferenceError: food2 is not defined
console.log(food3); // "Pizza"
```
When [**strict mode**](https://docs.microsoft.com/en-us/scripting/javascript/advanced/strict-mode-javascript) is enabled, `this` is undefined and will throw a `ReferenceError`. This is enabled by using the *directive prologue* `"use strict"`.
```javascript
"use strict";

function badCodeExample(){
    this.food1 = "Cheeseburger"; // TypeError: Cannot set property 'food1' of undefined
    var food2 = "Hotdog";
    food3 = "Pizza"; // ReferenceError: food3 is not defined
}
badCodeExample();
console.log(food1); // ReferenceError: food1 is not defined
console.log(food2); // ReferenceError: food2 is not defined
console.log(food3); // ReferenceError: food3 is not defined
```

<hr>

### 2. Object/Implicit
When the keyword `this` is inside of a declared object, the value of the keyword `this` will always be the closest parent object.
```javascript
var dog = {
    noise: "Woof!",
    color: "Brown",
    makeNoise: function(){
        console.log(this.noise);
    },
    getColor: function(){
        console.log(color);
    },
}
dog.makeNoise(); // Woof!
dog.getColor(); // ReferenceError: color is not defined
```

<hr>

### 3. Explicit Binding
Allows us to explicitly define the context of `this`.

<a name="dogs"></a>Consider the following code:
```javascript
var dog = {
    name: "Spot",
    breed: {
        name: "Dalmation",
        getName: function(){
            console.log(this.name);
        }
    },
    getName: function(){
        console.log(this.name);
    }
}
```
Because `dog` is an object, with other objects nested within it, the keyword `this` has a different *implicit* value, based on what the closest parent object is.
```javascript
dog.breed.getName(); // "Dalmation"
dog.getName(); // "Spot"
```
This may not be our desired functionality and we may want to *explicitly* define the context of `this`.

We can achieve this by using JavaScript's `call()`, `apply()` or `bind()` methods.

<hr>

### `call()` ###
Invokes a function, with an explicitly defined context for `this`.

- Syntax - `function.call(thisArg, arg1, arg2, ...)`
    - `thisArg` - The value of `this` provided for the call to a function.
    - `arg1, arg2, ...` - Optional. Arguments for the function.

We can fix the dogs example [above](#dogs) by using `call()` rather than invoking the methods directly;

```javascript
dog.breed.getName.call(dog); // "Spot"
dog.getName.call(dog); // "Spot"
```

`call()` is also used as a way of separating methods from properties and avoiding code duplication.

Consider the following:

```javascript
var food1 = { name: "pizza" };
var food2 = { name: "sandwich" };

var printFood = function(topping1, topping2, topping3) {
    console.log("Mmmmmh! A " + topping1 + ", " + topping2 + " and " + topping3 + " " + this.name);
}
```

We can call `printFood()` on our objects and pass in arguments by doing the following:

```javascript
printFood.call(food1, "cheese", "salami", "onion"); // "Mmmmmh! A cheese, salami and onion pizza"
printFood.call(food2, "bacon", "lettuce", "tomato"); // "Mmmmmh! A bacon, lettuce and tomato sandwich"
```

<hr>

### `apply()` ###
Almost identical to `call()` except rather than passing in comma-separated values as arguments, we pass in an array of arguments.

- Syntax - `function.apply(thisArg, [argsArray])`
    - `thisArg` - The value of `this` provided for the call to a function.
    - `arg1, arg2, ...` - An array-like object, specifying the arguments with thich the function should be called.

```javascript
printFood.apply(food1, ["cheese", "salami", "onion"]); // "Mmmmmh! A cheese, salami and onion pizza"
printFood.apply(food2, ["bacon", "lettuce", "tomato"]); // "Mmmmmh! A bacon, lettuce and tomato sandwich"
```

<hr>

### `bind()` ###

> "The `bind()` method creates a new function that, when called, has its this keyword set to the provided value, with a given sequence of arguments preceding any provided when the new function is called."
>
> -- [*MDN*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

<hr>

### `bind()` - basic example

Consider the following:

```javascript
var ryu = {
    name: "Ryu",
    moves: ["Hadouken", "Shoryuken", "Hurricane Kick"],
    win: function win() {
        var randomMove = this.moves[Math.floor(Math.random()*this.moves.length)];
        console.log(`${ this.name } wins with a devastating ${ randomMove }!`);
    }
}
```

We *could* do the following:

```javascript
ryu.win(); // "Ryu wins with a devastating Hurricane Kick!"
```

But say we just wanted to create a variable to reference the function, to be used later. We would get an error, because the context of `this` is lost.

```javascript
var ryuWin = ryu.win;
ryuWin(); // TypeError 
```

By using `bind()`, we can save the function for later use, along with conext for `this`.

```javascript
var ryuWin = ryu.win.bind(ryu); // "Ryu wins with a devastating Hadouken!"
```

<hr>

### `bind()` - `setTimeout()` example

> By default within `window.setTimeout()`, the `this` keyword will be set to the `window` (or `global`) object. When working with class methods that require `this` to refer to class instances, you may explicitly bind `this` to the callback function, in order to maintain the instance.
>
> -- [*MDN*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#With_setTimeout)

In the following example we would probably expect the console to print "*The dynamite exploded!!*"

```javascript
var bomb = {
    bombType: "dynamite",
    explode: function() {
        setTimeout(function(){
            console.log(`The ${this.bombType} exploded!!`);
        }, 1000)
    }
}
bomb.explode(); // "The undefined exploded!!"
bomb.explode.call(bomb); // "The undefined exploded!!"
```

The problem is that `setTimeout()` is actually called on the `window` (or `global`) object.

We can fix this by adding bind() to the end of the function definition, to explicitly define a context for `this`.

```javascript
var bomb = {
    bombType: "dynamite",
    explode: function(){
        setTimeout(function(){ // setTimeout is actually called on the window!
            console.log(`The ${this.bombType} exploded!!`);
        }.bind(this), 1000)
    }
}
bomb.explode(); // "The dynamite exploded!!"
```

<hr>

### `bind()` - partial application example
> "In computer science, partial application (or partial function application) refers to the process of fixing a number of arguments to a function, producing another function of smaller arity."
>
> -- [*Wikipedia*](https://en.wikipedia.org/wiki/Partial_application)

Example:
```javascript
let add = (a, b) => a+b;

let increment = add.bind(null,1);
let incrementBy2 = add.bind(null,2);

console.log(incrementBy2(3)); // 5
console.log(increment(3)); // 4
```

### 4. New

