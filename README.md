# Notes on "Javascript: Understanding the weird parts"

That document is what I wrote during following that amazing course by Anthony P.Alicea. You can find the [course](http://learnwebdev.net/) and the [first 3.5 hours](https://www.youtube.com/watch?v=Bv_5Zv5c-Ts).
This course is kinda old and isn't using ES6 stuffs but it is the most interesting piece of JS knowledge I found to that date.

These notes won't replace watching the course, they more of a personal reminder, for em and whoever watch the course.

## Things to know

 - javascript is synchronous and single-threaded.
 - objects and functions are closely related.
 - javascript doesn't care if arguments are not passed to function, they will just be setted to `undefined`
 - everything is an object or a primitive

### Execution context

The execution context is created in 2 phases: 
 - **creation phase**: Global object (variable environment), `this` and **outer environment** is setup in memory. Also setup the memory space for variables and functions (that step is called **hoisting**). An `argument` variable is also created for functions.
 - **execution phase**: We here have everything setup and it will run the code line by line.


### The scope chain
When a variable is called, javascript will does more than check the current **execution context**. An **execution context** have a reference to its outer environment. So in a **execution stack**,  the **outer environment** is not necessarily the **execution context** below itself.  It will instead depends on the **lexical environment**. So if my function is at the global level, the **outer environment** will be `global`. 

``` javascript
function a(){
  console.log(example)
}
function b(){
  var example = 1
  a()
}
var example = 2
b()

// => will return "2" because outer environment of a() is global even thought it's called in b() 

function b(){
  function a(){
    console.log(example)
  }
  var example = 1
  a()
}
var example = 2
b()
// => will return '1' because outer environment of a() is b() as a() sits in b() 

document.addEventListener('click', function() { console.log(this) });

// => will return the document object as functions are inheriting the execution scope

document.addEventListener('click', () => console.log(this));

// => will return the window object as arrow functions are inheriting the declaration scope
```

The fact of going down through these **outer environment** is called the **scope chain**.

### Functions are objects
A function is a special type of object. You can assign it a primitive, an object or a function as property. 
In a function there is also two important property: _name_ and _code_. A function without a name is called an anonymous function. 

### By value vs By reference
When passing a primitive to a variable, javascript will create a copy of that **value**, stored in its own memory.
``` javascript
a = 3
b = a
b
// => returns 3
a = 2
b
// => still returns 3
```

When passing an object (or a function as a function is an object), javascript will just **reference** to the object.
``` javascript
a = {name: "Tom"}
b = a
b
// => returns {name: "Tom"}
a.name = "Margaux"
b
// => returns {name: "Margaux"}
```

### `this`
`this` is setup during the creation phase of the **execution context**.

``` javascript
console.log(this)
// => returns the window object

function test(){
  console.log(this)
}
test()
// => returns the window object

var test = function(){
  console.log(this)
}
test()
// => returns the window object

var object = {
  name: "Tom",
  test: function(){
    console.log(this)
  }
}
object.test()
// => returns the object

var object = {
  name: "Tom",
  test: function(){
    function test(){
      console.log(this)
    }
    test()
  }
}
object.test()
// => returns the window object !!!
// for this case we usually need  a self = this
```

### Functional programming
Since javascript have **first class functions**, we are open to a lot of functional programming approaches.
Let's have an example:
``` javascript
var arr = [1,2,3]
function mapForEach(arr, fn){
  var newArr = []
  for(var i=0; i < arr.length; i++){
    newArr.push(fn(arr[i]))
  }
  return newArr
}

arr2 = mapForEach(arr, function(item){
  return item*10
})
console.log(arr2)
// => return [10,20,30]
```

### `undefined`
`undefined` is an internal value of javascript. It will be returned when a variable is initialized(via `var` or something). Different from `not defined` error that happen when the variable isn't in memory at all.

_Never set yourself a value to `undefined` yourself._ That would be valid but it's make the debugging flawy. 

### Coercion
The fact of changing a value to another type. The coercion is used when the equality operator is called. The coercion can act weird sometimes and that's why we will use triple equal `===` for a most "defined" behavior.

### Arguments
In a function javascript creates for us a variable called `arguments`. It's an array-like variable that we can use to access parameters like `arguments[0]`.

### Spread
An spread argument can be used for specify an undefined numbers of arguments. 
``` javascript
function test(arg1,arg2,...arg3){
  console.log(arg3)
}
test("Tom", "Quinonero")
// => will return []
test("Tom", "Quinonero", "Berlin", "JS Expert")
// => will return ["Berlin", "JS Expert"]
```

### Statement vs Expression
``` javascript
function test(){
  console.log("I'm a statement")
}

var test = function(){
  console.log("I'm an expression")
}
```

### IIFE: Immediate Invoked Functions Expressions
When calling an expression just after it been defined. 
``` javascript
var test = function(){
  console.log("test")
}()

(function(){
  console.log("test")
})()
```

### Closures
A feature of javascript. It's a combination of a function and it's lexical environment. The function will "remember" it's outer environment.

``` javascript
function buildFunction(){
  var arr = []
  for (var i = 0; i < 3; i++){
    arr.push(
      function() {
        console.log(i)
      }
    )
  }
  return arr
}

var fs = buildFunction()

fs[0]()
fs[1]()
fs[2]()
// => Return 3 3 3
// Because for doesn't create it's own execution environment 
// The outer environment will be the buildFunction and i is 3 in there
```

### `call()`, `apply()` and `bind()`
All functions have access to a `call()`, `apply()` and `bind()` function.
When creating an expression function, I can change what `this` is refering to using bind:
``` javascript
var person = {
  firstname: "Tom"
}
var test = function(salutation){
  console.log(salutation + ' ' + this.firstname)
}

test()
// => Will return undefined
var testBinded = test.bind(person)
testBinded()
// => Will return "tom"
```
I can also keep the same function using call:
``` javascript
test.call(person, "hi")
// => Will return "hi tom"
```
Apply can be used, the difference is that the arguments have to be wrapped into an array:
``` javascript
test.apply(person, ["hi"])
// => Will return "hi tom"
```
Cool patterns using these functions:
``` javascript
// Borowing a method
var post = {
  title: "How to be the best js person in the world",
  author: "Tom",
  getPostName: function(){
    return this.title + " by: " + this.author
  }
}

var post2 = {
  title: "How to be a bad js person",
  author: "Jack"
}

post.getPostName.bind(post2)()
// => Will return "How to be a bad js person by: Jack"
```
``` javascript
// function currying
function multiply(a,b){
  return a*b
}

// we set the function to be a copy of multiply with 2 as the first argument
var multiplyByTwo = multiply.bind(this, 2)

multiplyByTwo(4)
// => will return "8"
```

### Prototypal inheritance

In javascript the inheritance is prototypal. 
Each object have a `proto` property. If a property is not found on the object itself, it will look for it in the prototype of it. And that goes on. That is called the **prototype chain**.
All object inherits by default from the base object. That object is the base of every object.
All function inherits from the function base object. This object define property as arguments, bind, call, apply ... That's why every function can access these.
Same with arrays.

### The function constructor
We can build a object with a function constructor using `new` operator:
``` javascript
function Post(){
  this.title = "How to construct js objects"
  this.author = "Tom"
}

var post = new Post()
console.log(post)
// => Will return Post {title: "How to construct js objects", author: "Tom"}author: "Tom"title: "How to construct js objects"__proto__: Object
```
If the function doesn't specify a `return`, it's acting like we returned `this`.
A downside is that if we forgot to use the `new` keyword, it will not set the properties at the object level.
A convention for helping avoiding this issue is to start functions constructors name with a capital letter.

### Pure prototypal inheritance

``` javascript
var person = {
  firstname: "default",
  lastname: "default",
  greet: function(){
    return "Hi " + this.firstname + " " + this.lastname
  }
}

var tom = Object.create(person)
tom.greet()
// => will return "Hi default default"
tom.firstname = "Tom"
tom.greet()
// => will return "Hi Tom default"
```

### `class` operator
In javascript, since ES6, you can use `class` for for creating an object.
That works like a syntaxic sugar for constructing objects.
It has an `extends` possibility for using a prototype.


## Glossary 

 - **asynchronous**: Means more than one at a time. Since javascript is **synchronous**, the **event queue** is used to handle asynchronous stuffs. The asynchronous part will not happen in the javascript engine, it will mostly happens on the render engine of the browser or the HTTP engine.
 - **block**: A portion of code generally delimited by curly brackets `{}`. 
 - **callback function**: A function you give to another function to be executed when the other function is finished.
 - **coercion**: Converting a value to another type.
 - **dynamic typing**: Type is not defined explicitly, it's figured out when the code is running. It's the opposite of _static typing_.
 - **Event queue**: When an event happens, it's placed on the event queue. When the execution is finished (read the **execution stack** is empty), javascript will check periodically the event queue and check if something should be run in that case (eg: if an event is listened). If an event happens before the execution is done, javascript will only process this event when the execution is done.
 - **execution contexts**: A wrapper to help manage the code that is running. The default execution context is the global execution context. In a browser, the global object is the `window` object. `this` at the global level is the `window` object. _execution context != scope_
 - **execution stack**: A stack of **execution contexts**. When the script on the page is done, the stack is empty.
 - **expression**: A unit of code that results in a value.
 - **first class function**: Anything you can do with other types (assign to var, create them on the fly) can be done with functions. Functions in javascript are first-class functions.
 - **function currying**: Creating a copy of a function but with some preset parameters
 - **function constructor**: A function that is used to construct an object.
 - **hoisting**: Phenomenon that happens during the _creation phase_ of the **execution context**. It's setting up space for variables and functions into the memory. During this process, _all variables_ are initially set to `undefined` and functions are entirely wrote to the memory. So you can call a function that is defined later in the code event thought it's should not be done.
 - **inheritance**: One object gets access to the properties and methods of another object. Javascript uses prototypal inheritance.
 - **invocation**: Running a function. Invocation will create a new **execution context** and put it at the top of the **execution stack**. When the invocation is done, the **execution context** is removed form the **execution stack**.
 - **lexical environment**: Where something sits physically in the code you write 
 - **literal syntax**: A short syntax eg: `[]` for `new Array()`.
 - **mutation**: Changing something
 - **name/value pair**: A name which maps to a unique value. A name can only have one value in a given **execution context**.
 - **namespace**: A container for variables and functions.
 - **object**: A collection of **name/value** pair. An object can contains a primitive (called a property), another object (called also a property) or a function (called a method).
 - **operator**: An operator is a special function in javascript that use a different syntax. Its usually have two parameters and one result. The double equal comparator `==` is an operator. The plus `+` is an operator. The `.` is also an operator !
 - **operator associativity**: What order **operator** functions get called first when the operators have the same **precedence**. Left-to-right associativity or Right-to-left associativity. 
 - **operator precedence**: Define which **operator** function is called first. The _higher precedence win_. For example `*` has a higher precedence than `+`. 
 - **outer environment**: A reference to the environment where a function or a variable sits, it's **lexical environment**. By default (read "when declared at the root"), the outer environment is `global`. 
 - **primitives types**: The "default" types of javascript. Primitive type mean a single value, _not an object_. The six primitives types are: `undefined`, `null`, `boolean`, `number`, `string` and `symbol`(that is a ES6 type)
 - **reflection**: An object can look at itself, listing and changing its properties and methods.
 - **scope**: Where a variable is available in your code and if it's the truly same variable.
 - **scope chain**: The fact that a function or variable call not defined in the current **execution context** goes down to the **outer environment** for finding it.
 - **single threaded**: Mean that one command happen at a time.
 - **synchronous**: Mean that the code is happening one line at the time. Similar to **single threaded**.
 - **syntax parsers**: Program that reads the code and determinate what it does and if its syntax is valid. 
 - **variable environment**: Where the variable is. Since every function has it's own **execution context** a variable with the same name will have _no interactions_ between them. 
 - **whitespace**: Invisible character that create space in the code (spaces, tabs, returns).


## Resources

 - [Operator precedence on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)
 - [Equality comparison and sameness on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)
 - [Annotated underscore.js source code](https://underscorejs.org/docs/underscore.html)
