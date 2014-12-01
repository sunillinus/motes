#OOJS

## Scopes

### Lexical Scope

Scopes in your source code.

Lexical scopes consist of areas in your source code where a variable can be accessed with out any errors.

Global variables are in global scope and can be accessed from anywhere in your code.

```js
var hero = 'superman';
```

A new lexical scope is created eveytime a new function is defined - from letter 'f' of the function to the closing '}'.

```js
var foo = function() {
  var x = 'x';
};
```

If you assign a new variable with out declaring it - no `var` - makes it a global variable. **Confusing, don't do it**.

Looping or conditional blocks don't create a scope in JS.

## In-Memory Scopes or Execution Contexts

As a program runs, it creates in-memory data storage  for holding the variables and their values.

Build as the code runs, not when typed.

Every time a function is executed a new execution context (or in-memory scope) is created.

A new key value pair is added to the current execution context everytime the interpreter sees a variable assignment.

**Besides being key value pairs, in-memory scopes have nothing to do with in memory objects.**


## this

`o.fn()`: `this` -> o.

Inside a function, what `this` is bound to depends on how the function is invoked, not how it is defined or how it is looked up.

If the function is invoked as a free standing function, `this` is by default bound to the `<global>` object.

`setTimeout(cb, 1000)`: `this` inside `cb` when invoked by `setTimeout` will be bound to the `<global>` object because cb is invoked as a freestanding function: `cb()`.

`setTimeout(r.method, 1000)`: `this` inside `r.method` when invoked by `setTimeout` will be bound to the `<global>` object because r.method is invoked as a freestanding function inside setTimeout.

To set the right value for `this`, pass a wrapper function that invokes the method as a property:

```js
setTimeout(function() {
  r.method();  
}, 1000);
```

### Different ways `this` gets bound:

1. `this` inside a function is automatically bound to the object the function is called on
```js
o.fn(); // this -> o
```

2. When called as a freestanding function, `this` is by default bound to the `<globa>` object.
```js
fn(); // this -> <global>
```

3. You can use fn.call to explicitly set `this` inside a function:
```js
o.fn.call(o1); // this -> o1
o.fn.call(); // this -> <global>
```

4. If use `new` keyword, `this` is bound to a new object.
```js
new o.fn(); // this -> fn{}
```

## Prototype Chains

Mechanism to make an object behave as another object by delegating failed property lookups to the the other object. Better than duplicating properties since the delegation is done at look up time.

```js
var b = Object.create(a); // b uses a as it's prototype.
```

### Object Prototype
There is a top level object that every JS object delegates to - the Object prototype. Any failed property look ups, ultimately gets delegated to this Object.


#### Constructor Property
Tells us the contructor that was used to create a certain object. Since most objects don't have a `.constructor` property, the prototype chain is consulted which will usually end up in the `.constructor` property of the Object prototype which is the `Object() {}` function.

```js
var o = {};
var b = Object.create(o);
o.__proto__ // => Object
b.__proto__ // => a
o.constructor // function Object() {}
```

### Array Prototype
Arrays use an Array object `[]` as the prototype. Which in turn Object as it's prototype. But has it's own `constructor` property: `function Array() {}`

```js
var a = [];
a.__proto__ // => []
a.__proto__._proto__ // => Object
a.constructor // function Array() {}
```

## Object Decorator Pattern/Function
A function that takes an object as an argument and augments it with some properties. Example:

### 1st Form (with in-line methods)
```js
var carlike = function(obj, loc) {
  obj.loc = loc;
  obj.move = function() { // a new copy of the 'move' function is created in memory every time carlike is called.
    this.loc++;
  };
  return obj;
}
```

##### ^Refactored to use closure:
```js
var carlike = function(obj, loc) {
  obj.displacement = displacement;
  obj.move = function() {
    obj.loc++; //obj will be bound to the right object cause of closure
  };
  return obj;
}
```

```js
carlike({name: 'porsche'}, '911');
carlike({name: 'bmw'}, '2200');
```

### 2nd Form (with shared method):

```js
var carlike = function(obj, loc) { // decorator function
  obj.loc = loc;
  obj.move = move;
  return obj;
}

var move = function(car) { // only 1 copy of the 'move' function in the memory.
  car.loc++;
};
```



## Classes
- A Class are category of similar objects.
- Contructor functions are functions that create a new instance of a class.
- Conventional to use a captilazed noun as the name of the class/contructor function.
- In JS, classes are essentially functions that are capable of creating similar objects.

## Functional Class Pattern (with in-line methods)
- Unlike a decortor pattern, the functional class pattern builds the object it self while a decorator pattern accepts it as an argument.

```js
var Car = function(loc) {
  var obj = {loc: loc};
  obj.move = function() {
    obj.loc++;
  };

  return obj;
};

var bmw = Car(3);
bmw.move();
var porsche = Car(911);
porsche.move();
```

^This leads to a new function object being created for every car instance that is created. To avoid this put it in the global/outer scope or:

## Functional Class Pattern (with Shared Methods  or Functional Shared Pattern)

```js
var Car = function(loc) {
  var obj = {loc: loc};
  obj.move = move;

  return obj;
};

var move = function() {
  this.loc++; // 'this' will automatically be bound to the correct object
};
```

^ Refactoring it further by adding the shared methods programmatically from another object:

```js
var Car  = function(loc) {
  var obj = {loc: loc};
  extend(obj, methods); // extend is not native js function

  return obj;
};

var methods = {
  move: function() {
    this.loc++;
  }
};
```

^ Using non-global variable to store the methods and also encapsulating it with Car function object:

```js
var Car = function(loc) {
  var obj = {loc: loc};
  extend(obj, Car.methods);

  return obj;
};

Car.methods = {
  move: function() {
    this.loc++;
  }
};
```

>^Remember functions are specialized objects (in that it can be invoked) and can store properties just like other objects. Invoking a function executes the functions body and has no interaction with it's properties.

## Prototypal Class Pattern

Similar to functional shared class pattern, but instead of copying the methods over to each instance of the class (using `extend`), use prototype object to store the shared methods and make the instances delgate to that shared prototype object:

```js
var Car = function(loc) {
  var obj = Object.create(Car.methods);
  obj.loc = loc;

  return obj;
};

Car.methods = {
  move: function() {
    this.loc++;
  }
};

```

**^Since this pattern (building a holder object for methods and attaching it as a property to class constructor function) is so common, the language has official conventions to support this pattern. So whenever a function is created, it will automatically have an object attached to it that you can use to hold methods in case you use that function as a constructor. This method container object that comes with every function is stored at the key `prototype`.** So using this default object at prototype:

```js
var Car = function(loc) {
  var obj = Object.create(Car.prototype);
  obj.loc = loc;

  return obj;  
};

Car.prototype.move = function() {
  loc++;
};

var bmw = Car(3);
```
** Nothing special about the `prototype` key. Can just as well be `blah` key** So:
1. Car function object won't delegate it's failed lookups to Car.prototype.
2. Only reason the instances created by the Car constructor delegate it's failed lookups to Car.prototype os because of the `Object.create(Car.prototype)` call.
3. So above the `bmw` object's prototype is Car.prototype, but Car's prototype is not Car.prototype, but some other object used by all function objects (`function Empty() {}`).
4. The object at `.prototype` is just a freely provided object with every function for storing things but with no special characteristics (except the `.constructor` property)

### .constructor property
The prototype object has a special `constructor` property that points back to the function it belongs to. so `Car.prototype.contstructor === Car` will be true.

So any instance can look up the constructor that created it using the `constructor` property (provided the Constructor used Object.create() to set the instance's prototype to the Constructor's `.prototype`. Now the object will delegate the '.constructor' to the prototype which in turn will point back to the constructor).

### instanceof operator
Checks if right operands prototype property exists anywhere in the prototype chain of the left operand.

`bmw instanceof Car` checks if Car's prototype property exists in the prototype chain of the bmw.


## Pseudoclassical Pattern
- Attempts to represent class system from other languages (like Java) by adding some syntactic sugar to Prototypal class pattern.
- You use the keyword `new` before invoking a function. This will cause the function to run in a 'constructor' mode and the interpreter will insert some code at the begining and end of your function:
  - `this = Object.create(Xyz.prototype);` at the beginning
  - `return this;` at the end.
- JS engines apply some optimizations for the pseudoclassical pattern.

So:
```js
var Car = function(loc) {
  var obj = Object.create(Car.prototype);
  obj.loc = loc;

  return obj;
};

Car.prototype.move = function() {
  this.loc++;
};

var bmw = Car(3);
```

Can be refactored to:

```js
var Car = function(loc) {
  // this = Object.create(Car.prototype); <- inserted by interpreter behind the scenes
  this.loc = loc;
  // return this <- inserted by interpreter behind the scenes
};
Car.prototype.move = function() {
  this.loc++;
};

var porsche = new Car(911);
```


## Summary of Different Class Patterns in JS

### 1. Functional Class pattern with In-line Methods
```js
var Car = function(loc) {
    var obj = {loc: loc};
    obj.move = function() {
      loc++;
    };

    return obj;
};

var porsche = Car(911);
```

### 2. Functional Class Pattern with Shared Methods
```js
var Car = function(loc) {
  var obj = {loc: loc};
  obj.move = move;

  return obj;
};

var move = function() {
  this.loc++;
};

var porsche = Car(911);
```

### 3. Prototypal Class Pattern
```js
var Car = function(loc) {
  var obj = Object.create(Car.prototype);
  obj.loc = loc;

  return obj;
};

Car.prototype.move = function() {
  loc++;
};

var porsche = Car(911);
```

### 4. Pseudoclassical Pattern
```js
var Car = function(loc) {
  this.loc = loc;
}

Car.prototype.move = function() {
  loc++;
};

var porsche = new Car(911);
```

** Except functional class pattern with inline methods, all the other patterns have 2 distinct secions:
1. The constructor part holds any properties that are distinct among the instances of the class
2. The common shared properties.


## Superclasses and Subclasses

### Functional Subclasses

```js
var Car = function(loc) {
  var obj = {loc: loc};
  obj.move = function() {
    obj.loc++;
  };

  return obj;
};

var Taxi = function(loc) { // subclasses Car
  var obj = Car(loc);
  obj.call = funntion() {
    //...
  };

  return obj;
};

var myCar = Car(5);
var cab1 = Taxi(10);
```

### Pseudoclassical Subclasses

```js
var Car = function(loc) {
  this.loc = loc;
};

Car.prototype.move = function() {
  this.loc++;
};

var Taxi = function(loc) {
  // this = Object.create(Taxi.prototype); <= interpreter does this
  Car.call(this, loc); // executes Car constructor with 'this' from the above line
}
// Taxi.prototype = new Car(); X Wrong!! Creates an unwanted and unnecessary new instance of Car (whihc might fail!) and delegates Taxi.prototype to Car.prototype through that new instance of Car.
Taxi.prototype = Object.create(Car.prototype);
Taxi.prototype.constructor = Taxi; // Have to set this explicitly since we are overwriting the automatically generated prototype object (which has the constructor property) in above line.
Taxi.prototype.call = function() {
  //...
};

var myCar = new Car(5);
var cab1 = new Taxi(10);
```

#### 3 Steps:

1. Call superclass in the subclass constructor with .call passing the subclass 'this': `Car.call(this);`
2. Set the subclass prototype property to an object that delegates to the superclass prototype object: `Taxi.prototype = Object.create(Car.prototype);`
3. Set the constructor property of the new object created above to the subclass constructor function. `Taxi.prototype.constructor = Taxi;`. Have to set this explicitly since we are overwriting the automatically generated prototype object (which has the constructor property) in above line. With out it, a new instance of Taxi will have Car as it's .constructor instead of Taxi.
