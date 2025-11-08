# Fundamentals

- In 1995, JavaScript was developed by Brendan Eich, a Netscape developer. Initially named Mocha, it was later renamed to LiveScript.
- Three main parts in JS - **ECMAScript | DOM | BOM**

## Variables
- Variables cannot start with digit,special characters
- camelCase by convention
- Dynamically typed (don't need to explicitly specify the variable's type)
- Examples for static-typed languages : Java/ C#.
- Accessing undeclared variables gives a ReferenceError
- Declaring and initializing a const must be done in a single statement since const variables holds a value that doesn't change
- Has 2 data types, **Primitive, complex**
**Primitive**
    - undefined --> **console.log(typeof undeclaredVar); // undefined**
    - null --> **console.log(null == undefined); // true**
    - NaN --> stands for Not a Number. It is a special numeric value that indicates an invalid number, **console.log('a'/2); // NaN**;
                    NaN does not equal any value, including itself, so **NaN == NaN returns false**.
    - string --> JavaScript **strings are immutable**, This means that modifying a string always results in a new string,
                      leaving the original string unchanged.
    - boolean-->
    - symbol --> let s1 = Symbol(); //The Symbol function creates a new unique value every time you call it.
                        **console.log(Symbol() == Symbol()); // false**
    - bigint -->

**Complex**
    - Object --> collection of properties, where each property is defined as a key-value pair.
      - Property can be accessed using both don and array notations (. || []),
      - If a property name has whitespace (bad practice), you must access it with array notation
      - Deleting a property --> delete objectName.propertyName.
      - Checking if a property exists --> propertyName in objectName

## Numeric Separator
- Numeric separators ( \_ ) --> let amount = 120_201_123.05; // 120201123.05
- Can be used for any number type(integer,factional and exponent, floating, BigInt,binary,octal,hex)
- It's important to note that all numbers in JavaScript are floating-point numbers.

## Boolean type
- Boolean() function casts the values of other types to boolean values:
- Boolean() return true for any of -> (Any non-empty string, Any Non-zero number,Any object)
- Boolean() return false for any of ->(empty string,0, NaN,null,undefined)

## String
- Template literal and string interpolation --> **console.log(Hi, I'm ${name}.)**
- To access the last character of the string --> **console.log(str[str.length -1]) // first --> str[0] OR str.charAt(0)**
- Converting values to string --> String(n); | ” + n | n.toString() --> toString() method doesn't work for undefined and null

## Primitive vs. Reference Values
- Static data is the data whose **size is fixed** at **compile time**
- Primitive values && Reference values that refer to objects (this is not object, but reference).
- Since primitive data types has fixed size, JS store them in stack by allocating fixed amount of memory space
- Objects (and functions) on the heap , doesn't allocate a fixed amount of memory
- When you assign a primitive value from one variable to another, the JavaScript engine
  creates a copy of that value and assigns it to the variable.
  **changing a value of one variable, won't affect the other.**

![Stack-and-heap](./images/Js-stack-and-heap.png)

## Arrays

- Note that if you use the **Array()** constructor to create an array and pass a **single number** into it, you are creating an array with an initial size.
- Appending an element --> array.push
- Adding an element to the beginning --> array.unshift(1)
- Removing an element from the beginning → array.shift();
- Finding an index of an element → array.indexOf(1);
- Check if a value is an array → Array.isArray(seas)

# Operators

- In (- , \*, /) If either value is not a number, the JavaScript engine implicitly converts it into a number using the **Number()** function.
- If the value is an object, Js use object”s **valueOf** function, if it is not exists Js use **toString** function of the object.
- If you want to assign a single value to multiple variables, you can chain the ->

```javascript
let a = 10,
	b = 20,
	c = 30;
a = b = c;
// all variables are 30 JavaScript evaluates from right to left
```

- When you apply the unary plus/minus (+ val / -val) operator to a non-numeric value, it performs a number conversion using the **Number()**

```javascript
let s = "10";
console.log(+s); // 10
let f = false,
	t = true;
console.log(+f); // 0 console.log(+t); // 1
```

- If it is an object, it uses **valueOf/toString**
- The prefix increment operator adds one to a value. The value is changed before the statement is evaluated.
- The postfix increment operator adds one to a value. The value is changed after the statement is evaluated.
- The prefix decrement operator subtracts one from a value. The value is changed before the statement is evaluated.
- The postfix decrement operator subtracts one from a value. The value is changed after the statement is evaluated.

```javascript
let weight = 90; 
let newWeight = weight++ + 5; 
console.log(newWeight); // 95 console.log(weight); // 91
//Adds one to the value after evaluating the statement

let weight = 90; 
let newWeight = ++weight + 5; 
console.log(newWeight); // 96 console.log(weight); // 91
//Adds one to the value before evaluating the statement
```

# Control flow Statements
- if, if...else, if...else...if, Ternary Operators, switch, while,do...while,for loop, break,continue,Comma operator

# Functions
- Every function in JavaScript implicitly returns **undefined** unless you explicitly
- Inside a function, you can access an object called **arguments** that represents the named arguments of the function.
- The **arguments** object behaves like an [array](https://www.javascripttutorial.net/javascript-array/) though it is not an instance of the [Array](https://www.javascripttutorial.net/javascript-array/) type.
- The value of the **arguments** object inside the function is the number of actual arguments that you **pass to the function**

```javascript
function add(x, y = 1, z = 2) {
    console.log( arguments.length );
    return x + y + z;
}

add(10); // 1
add(10, 20); // 2
add(10, 20, 30); // 3
```
- Variable hoisting means the JavaScript engine moves the variable declarations to the top of the script, this behave differently for **var** and **let**

 ```javascript
  console.log(counter);// 👉 undefined
  var counter = 1;

  console.log(counter); //"ReferenceError: Cannot access 'counter' before initialization
  let counter = 1;
```
- During the creation phase of the global execution context, the JavaScript engine places the variable **counter** in the memory and initializes its value to **undefined**.
- JavaScript engine hoists the variable declarations that use the **let** keyword. However, it doesn't initialize the **let** variables.
- The JavaScript engine doesn't hoist the function expressions, arrow functions and class expressions.
    
- Function hoisting is a mechanism in which the JavaScript engine physically moves function declarations
    to the top of the code before executing them.
- In JavaScript, functions are first-class citizens, meaning they can be stored in variables, passed
    as arguments, and returned from other functions.
- In JS, arguments are passed by value by default, so changes to arguments do not reflect outside,
    like pass by reference; pass by reference is not available in JS.
- But, if you pass a non-primitive value (object), then it refers to the same value, so changes
    to that will be reflected outside of the function.

```javascript
let person = {
	name: "John",
	age: 25,
};

function increaseAge(obj) {
	obj.age += 1;

	// reference another object
	obj = { name: "Jane", age: 22 };
	//After the function finishes execution, **obj** (a local variable) is discarded.
}

increaseAge(person);
console.log(person);
console.log(obj); // This returns a ReferenceError, because function execution is completed at this level
```

![Pass-by-value-object](./images/Js-pass-by-value-object.png) ![Pass-by-value-object-and-change-reference](./images/Js-pass-by-value-object-and-change-reference.png)

```javascript
const styles = { color: "red" };
function change(styles) {
	styles = { background: "black" }; //Use the same name, but this is a local variable and discard it after the execution.
}
change(styles);
console.log(styles); //{ color: 'red', }
```
- In JavaScript, a parameter has a default value of [undefined](https://www.javascripttutorial.net/javascript-data-types/#undefined). (we pass arguments into functions, and functions accept them as parameters)
- In JavaScript, you **cannot directly use named parameters**. instead you have to pass them like **undefined**
**createDiv(undefined, undefined, 'solid 5px blue');**
- The most effective way to handle this type of situation is by passing or accepting the parameters as an object.
**createDiv({ border: 'solid 5px blue' });**

# Objects & Prototypes
## Constructor functions

```javascript
function Person(firstName, lastName) { //Note Uppercase P
    // this = {}; initially this empty object is created
    this.firstName = firstName;
    this.lastName = lastName;

    this.getFullName = function () {
        return this.firstName + " " + this.lastName;
    };
    // return this; finally this OBJECT is retured
}
let person = new Person("John", "Doe"); //Note the new keyword
console.log(person.getFullName());//John Doe
```
- The problem with the constructor function is that when you create multiple instances of the **Person**, the **this.getFullName()** is duplicated in every instance, which is not memory efficient.
- To resolve this, you can use the [prototype](https://www.javascripttutorial.net/javascript-prototype/) so that all instances of a custom type can share the same methods.
- Constructor function can be called without the new keyword, like
  **let person = Person('John','Doe');**
- But, The **Person** just executes like a regular function. Therefore, the **this** inside the **Person** function doesn't bind to the **person** variable but the [global object](https://www.javascripttutorial.net/es-next/javascript-globalthis/).
**console.log(person.firstName); //TypeError: Cannot read property 'firstName' of undefined**

```javascript
function Person(firstName, lastName) {
    if (!new.target) {
        return new Person(firstName, lastName);
    }

    this.firstName = firstName;
    this.lastName = lastName;
}
let person = Person("John", "Doe");
console.log(person.firstName);
```

```javascript
function Person(name) {
    this.name = name;
}
//call as a constructor
let john = new Person('John');//define type using new keyword and 'this' refers to local context 
console.log(john.name); // john 

Person('Lily'); //this will call Person as a regular function and 'this' refers to globalThis
console.log(globalThis.name);//Lily

//using new.target to ensure Person is called with new keyword.
function Person(name) {
    if (!new.target) { //undefined if called normally, else return a reference to the constructor.
        throw "must use new operator with Person";
    }
    this.name = name;
}
let john = new Person('John');
console.log(john.name); // john
Person('Lily'); //Uncaught must use new operator with Person
```

## Object prototype

- Every object has its own property called a **prototype**, objects can inherit features from one another via **prototypes**
- Because the **prototype** itself is also another object, the **prototype** has its own **prototype**.
- This creates a something called **prototype chain**. The prototype chain ends when a prototype has **null** for its own prototype.
- If you access a property that doesn't exist in an object, the JavaScript engine will search in the prototype of the object
- If the JavaScript engine cannot find the property in the object's prototype, it'll search in the prototype's prototype until it finds the property or reaches the end of the prototype chain
- Object() is a FUNCTION
- Object.prototype is a OBJECT
- Object.prototype.constructor property references the **Object** function
**console.log(Object.prototype.constructor === Object); // true**

```javascript
function Person(firstName, lastName) { //A Constructor function
    this.firstName = firstName;
}
console.log(Person.prototype.constructor === Person) //true
let p = Person('Alex'); //if we call without new keyword, the constructor function behaves like a regular function.
```
- JavaScript links the **Person.prototype** object to the **Object.prototype** object via the **[[Prototype]]**, which is
  known as a **prototype linkage**.

```javascript
Person.prototype.greet = function() {
      return "Hi, I'm " + this.name;
  }
  let p1 = new Person('John');
  let greeting = p1.greet();  
  
  let p2 = new Person('Jane');
  let greetingP2 = p2.greet();
  
  console.log(greeting); //Hi, I'm Jhone
  console.log(greeting); //Hi, I'm Jane

  //In ES6 since we have class keyword, we don't have to do this to define a custom type

  console.log(Person.prototype === Object.getPrototypeOf(p1)); //true
  console.log(Person.prototype === p1.constructor.prototype); // true
```

![ProtoType-linkage](./images/Js-protoType-linkage.png)

**let person = { name: "John Doe", greet: function () { return "Hi, I'm " + this.name; } };**

- The person object has a link to the anonymous object referenced by the **Object()** function. The **[[Prototype]]** represents the linkage:
- **person** object can call any methods defined in the anonymous object referenced by the **Object.prototype** **console.log(person.toString());**
- Since JavaScript engine cannot find it on the **person** object. Therefore, it follows the prototype chain and searches for the method in the **Object.prototype** object
- `console.log(person.__proto__ === Object.prototype); // true`

## prototypal inheritance

```javascript
let person = { name: "John Doe", greet: function () { return "Hi, I'm " + this.name; } };

let teacher = {
    teach: function (subject) {
        return "I can teach " + subject;
    }
};

teacher.__proto__ = person;
console.log(teacher.greet());
/*Since the JavaScript engine cannot find the method in the **teacher** object, it follows the prototype chain and searches for the method in the **person** object. Because the JavaScript engine can find the **greet()** method in the **person** object, it executes the method.
**teacher** object inherits the methods and properties of the **person** object
This kind of inheritance is called prototypal inheritance.
*/
```

- prototypal inheritance  in ES5**let teacher = Object.create(person);**
- **console.log(Object.getPrototypeOf(teacher) === person); //true**
- Prototypal inheritance in ES6 is using classes and is the recommended way.
- JavaScript objects have two types of properties: **data** properties and **accessor** properties.
- JavaScript uses internal attributes denoted **[[...]]** to describe the characteristics of properties.
- **data** properties have 4 attributes **[[Configurable]]**, **[[Enumerable]]**, **[[Writable]]**.
- **accessor** properties have 4 attributes  and **[[Configurable]]**, **[[Enumerable]]**, **[[Get]]**, and **[[Set]]**
- A property can be defined directly on an object or indirectly via the **Object.defineProperty()** or **Object.defineProperties()** methods. These methods can be used to change the attributes of a property.

```javascript
var product = {};

Object.defineProperties(product, {
    name: { value: 'Smartphone', enumerable: true },
    price: { value: 799 },
    tax: { value: 0.1 },
    netPrice: {
        get: function () {
            return this.price * (1 + this.tax);
        }
    }
});

console.log('The net price of a ' + product.name + ' is ' + product.netPrice.toFixed(2) + ' USD');
//The net price of a Smartphone is 878.90 USD
//If you want to loop through using for...in when defining properties using defineProperty/defineProperties,
//you have to manually set enumerable: true 
//above print only name
```
- The **for...in** loop iterates over the **enumerable properties** of an object. It also goes up to the [prototype](https://www.javascripttutorial.net/javascript-prototype/) chain and enumerates inherited properties.
- Avoid using **for...in** loop to iterate over elements of an array, especially when the index order is important.
- A property is enumerable if it has the **enumerable** attribute set to **true**. The **obj.propertyIsEnumerable()** determines whether or not a property is enumerabl
- A property created via a simple assignment or a property initializer is enumerable.
- A property that is directly defined on an object is an own property.
- The **obj.hasOwnProperty()** method determines whether or not a property is own. //**console.log(product.hasOwnProperty('ssn')); // => false**
- Computed properties allow you to use the values of expressions as property names of an object.
- 
```javascript
let propertyName = 'dynamicPropertyName'; 
const obj = { [propertyName] : value }
obj.dynamicPropertyName

const createObject = (key, value) => { 
    return { [key]: value, };
};
const person = createObject('name', 'John'); 
console.log(person); //{ name: 'John' }
```

# Classes (ES6 - 2015)
## private properties/methods static properties/methods

```javascript
class Person {
  static count = 0;
  #firstName; //define private properties
  #lastName;
  constructor(firstName, lastName) {
    Person.count = 1; //or this.constructor.count = 1 to access static properties
    this.#firstName = Person.#validate(firstName); //access static methods
    this.#lastName = Person.#validate(lastName);
  }
  getFullName(format = true) {
    return format ? this.#firstLast() : this.#lastFirst();
  }
  static #validate(name) {
    if (typeof name === 'string') {
      let str = name.trim();
      if (str.length >= 3) {
        return str;
      }
    }
    throw 'The name must be a string with at least 3 characters';
  }

  #firstLast() {
    return **${this.#firstName} ${this.#lastName}**;
  }
  #lastFirst() {
    return **${this.#lastName}, ${this.#firstName}**;
  }
}

let person = new Person('John', 'Doe');
console.log(person.getFullName());
```

# Advanced Functions
- In JavaScript, all functions are objects and  they are instances of the **Function** type
- Typically, a local variable only exists during the function's execution.
- A closure is a function that preserves the outer scope in its inner scope.

```javascript
for (var index = 1; index <= 3; index++) {
    setTimeout(function () {
        console.log('after ' + index + ' second(s):' + index);
    }, index * 1000);
}
after 4 second(s):4
after 4 second(s):4
after 4 second(s):4
//By the time the first callback executes (after 1000ms), the loop has already completed, and **index** is **4**.
//Fix 1: use IIFE for versions older that ES6 (immediately-invoked-function-expression)
//Fix 2: use let keyword in ES6/Lexical scoping
```
```javascript
function Car() {
  this.speed = 0;

  this.speedUp = function (speed) {
    this.speed = speed;
    setTimeout(function () {
      console.log(this.speed); //undefined
    }, 1000);
  };
}

let car = new Car();
car.speedUp(50);

//undefined, inside an anonymous function, **this** doesn't inherit the **this** value from the surrounding method. 							            
//Instead, it defaults to the global object (**window** or **global**) in non-strict mode or **undefined** in strict mode.
//to fix this, use arrow functions OR assign **this** to a variable. let self=this and self.speed
```
- An arrow function doesn't have the **arguments** object.
```javascript
function show() {
  return (x) => x + arguments[0];
}

let display = show(10, 20);
let result = display(5);
console.log(result); // 15, this **arguments** object belongs to the **show()** function, not the arrow function.
```
- An arrow function doesn't have its binding to **this** or **super**. they inherit **this** from the parent scope.
- An arrow function doesn't have **arguments** object, **new.target** keyword, and **prototype** property.
- You should not use it as an event handler, a method of an object literal, a prototype method, or when you have a function that uses the **arguments** object.
- A callback is a function passed into another function as an argument to be executed later.
- A high-order function is a function that accepts another function as an argument.

# Promises & Async/Await
```javascript
function getUsers() { 
    return new Promise((resolve, reject) => {
        //Code to be executed. EX) API calld, DB calls
    })
}

function onFulfilled(users) { console.log(users); } 
function onRejected(error) { console.log(error); }

getUsers().then(onFulfilled, onRejected);

//OR
promise.then(
    (users) => console.log, 
    (error) => console.log 
);
//ALL TOGETHER
getUsers()
  .then((users) => {
    console.log(users);
  })
  .catch((error) => {
    console.log(error);
  })
  .finally(() => {
    render();
  });
```
- Use **then()** method to schedule a callback to be executed when the promise is fulfilled, and **catch()** method to schedule a callback to be invoked when the promise is rejected.
- Place the code that you want to execute in the **finally()** method whether the promise is fulfilled or rejected.
- **Promise.resolve('Success').finally(() => console.log('Done')); //Done**
- **Promise.resolve('Success').then(console.log); //Success**
- **Promise.reject('Error').catch(console.log);//Error**
- ES2017 introduced the [**async**/**await**](https://www.javascripttutorial.net/es-next/javascript-async-await/) that helps you write code that is cleaner than using the promise chaining technique.
- The **Promise.all()** method accepts a list of promises and returns a new promise that resolves to an array of results of the input promises if all the input promises are resolved, or rejected with an error of the first rejected promise.
- **Promise.all([Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)]).then(console.log); //[1, 2, 3]**
- **Promise.all([Promise.resolve(1), Promise.reject('Error'), Promise.resolve(3)]).catch(console.log); //Error**
- The **Promise.race(iterable)** method returns a new promise that fulfills or rejects as soon as one of the promises in an iterable fulfills or rejects, with the value or error from that promise.
- **Promise.race([Promise.resolve(1), new Promise(resolve => setTimeout(() => resolve(2), 1000))]).then(console.log); //1**
- If one of the promises in the iterable object is fulfilled, the **Promise.any()** returns a single promise that resolves to a value which is the result of the fulfilled
- The **Promise.any()** returns a promise that is fulfilled with any first fulfilled promise even if some promises in the iterable object are rejected
- **Promise.any([Promise.reject('Error1'), Promise.resolve('Success1'), Promise.resolve('Success2')]).then(console.log).catch(console.error); //Success1**
- **Promise.any()** waits for the first promise to fulfill (resolve), whereas **Promise.race()** waits for the first promise to settle (resolve or reject).
- **Promise.any([Promise.reject('Error1'), Promise.reject('Error2')]).then(console.log).catch(error => console.log(error.errors));**
- Since all promises reject, **Promise.any()** rejects with an AggregateError. The **errors** property of this AggregateError contains an array of rejection reasons: ['Error1', 'Error2'].
- The **Promise.allSettled()** method accepts an iterable of promises and returns a new promise that resolves when every input promise has settled with an array of objects that describes the result of each promise in the iterable object.
- Before ES6, we had to use callbacks for handling asynchronous programming in JavaScript. However, this approach often led to **callback hell** when the number of nested functions grew.
- ES6 introduced the Promise object, which made asynchronous programming more manageable by allowing chaining.
- Then, in ES2017, the **async and await** keywords were introduced to further simplify asynchronous code, making it cleaner and easier to read.
- async/await is essentially syntactic sugar built on top of Promises. When we use the await keyword, the function returns a Promise object, making it possible to work with asynchronous operations in a more readable, synchronous-like manner.
- 
```javascript
async function showServiceCost() {
    try {
       let user = await getUser(100);
       let services = await getServices(user);
       let cost = await getServiceCost(services);
       console.log(**The service cost is ${cost}**);
    } catch(error) {
       console.log(error);
    }
}
```

# Iterators
- ES6 provides built-in iterators for the collection types **Array**, **Set**, and **Map**, We can use a **for...of** loop to iterate over an iterable object.
- If you have a custom type and want to make it iterable so that you can use the **for...of** loop construct, you need to implement the [iteration protocols](https://www.javascripttutorial.net/javascript-iterator/)
- A generator can pause midway and then continues from where it paused
```javascript
function* generate() {
    console.log('invoked 1st time');
    yield 1; //the **yield** statement returns a value and pauses the execution of the function.
    console.log('invoked 2nd time');
    yield 2;
}
```
- A generator returns a **Generator** object without executing its body when it is invoked.
- A **Generator** object is [iterable](https://www.javascripttutorial.net/es6/javascript-iterator/). So you can use the **for...of** loop.
- Use cases of **for...of** loop

```javascript
// Iterating over arrays

let scores = [80, 90, 70];

for (let score of scores) {
    score = score + 5;
    console.log(score);
}

let scores = [80, 90, 70];

for (const score of scores) {
    console.log(score);
}

let colors = ['Red', 'Green', 'Blue'];

for (const [index, color] of colors.entries()) {
    console.log(**${color} is at index ${index}**);
}

//In-place object destructuring with for…of

const ratings = [
    {user: 'John',score: 3},
    {user: 'Jane',score: 4},
    {user: 'David',score: 5},
    {user: 'Peter',score: 2},
];

let sum = 0;
for (const {score} of ratings) {
    sum += score;
}

console.log(**Total scores: ${sum}**); // 14

//Iterating over strings
let str = 'abc';
for (let c of str) {
    console.log(c);
}

//Iterating over Map objects
let colors = new Map();

colors.set('red', '#ff0000');
colors.set('green', '#00ff00');
colors.set('blue', '#0000ff');

for (let color of colors) {
    console.log(color);
}

//Iterating over set objects
let nums = new Set([1, 2, 3]);

for (let num of nums) {
    console.log(num);
}
```

- The **for...in** iterates over all **enumerable properties of an object**. It doesn't iterate over a collection such as **Array**, **Map** or **Set**.

# Modules
- A module is a JavaScript file that executes in **strict mode**. It means that any **variables** or **functions** declared in the module won't be added automatically to the **global scope**.
- Use the **export** statement to export variables, functions, and classes.
- Use the **import** statement to import variables, functions, and classes from other modules.
- Use **type="module"** in the script tag to instruct the web browser to load a JavaScript file as a module.

```javascript
//lib.js
function display(message) {...}
export { display };

//When exporting a value using a named export, you can rename it like this
export { display as MyDisplay };

//index.js
import { display } from './lib.js';

//modules that import the value may also rename it when importing
import { display as Mydisplay } from './lib.js';


//index.html
<script src="js/index.js" type="module"></script>
```
- A module can have **multiple named exports** but **only one default export**.
- It's possible to combine the variable/function/class declaration, and export in a single statement.
- Exporting multiple variables **export { MIN, MAX, count };**.
- When importing a default export, you don't need to place the variable inside curly braces.
- When importing a default export, you can assign it any names you want when you import it.

```javascript
let message = 'Hi';
export { default as message };
import message from 'module.js';

```
- Module support Re-exporting
- Namespace import
```javascript
import * as name from 'module.js';
name.myVariable
name.myFunction
name.myClass
name.default //If the importing module has a default export

//You cannot access default exports using it's name

export default function sayHi() {
  alert('Hi');
}
import * as greeting from './greeting.js';
greeting.sayHi(); //Uncaught TypeError: greeting.sayHi is not a function
```
- The **import()** allows you to dynamically import a module when needed.
- The **import()** returns a **Promise** that will be fulfilled once the module is loaded completely.
- Since the import() returns a Promise, you can use the **async/await**.
- To load multiple modules dynamically, you can use the **Promise.all()** method.

```javascript
const btn = document.querySelector('.btn');

btn.addEventListener('click', () => {
  import('./greeting.js')
    .then((greeting) => {
      greeting.sayHi();
    })
    .catch((error) => {
      console.error(error);
    });
});
```

## A top-level await module
- A top-level await module acts like an **async** function.

```javascript
//index.html uses app.mjs, app.mjs uses user.mjs, user.mjs calls and API and fetch Json data

//index.html
<script type="module" src="app.mjs"></script>

//user.mjs
const url = 'https://jsonplaceholder.typicode.com/users';
const response = await fetch(url);
let users = await response.json();
export { users };

//app.mjs
import { users } from './user.mjs';
function render(users) {...}

let container = document.querySelector('.container');

try {
  container.innerHTML = render(users);
} catch (error) {
  container.innerHTML = error;
}
```
# Symbol

- ES6 added **Symbol** as a new primitive type
- **let s = Symbol('foo');**
- The **Symbol()** function creates a new **unique** value each time you call it

```javascript
let firstName = Symbol('first name');
let firstName2 = Symbol('first name');
console.log(firstName, firstName2, firstName === firstName2);
//Symbol(first name) Symbol(first name) false

console.log(typeof firstName); // symbol

let s = new Symbol(); // error, Since a symbol is a primitive value
```

- ES6 provides you with a global symbol registry that allows you to share symbols globally
```javascript
let ssn = Symbol.for('ssn');
let citizenID = Symbol.for('ssn');
console.log(ssn === citizenID); // true
console.log(Symbol.keyFor(citizenID)); // 'ssn'
console.log(Symbol.keyFor(systemID)); // undefined
```

## Well-known symbols
- Symbol.hasInstance
- Symbol.iterator
- Symbol.isConcatSpreadable
- Symbol.toPrimitive

# Map Object
```javascript
let john = {name: 'John Doe'},
    lily = {name: 'Lily Bush'},
    peter = {name: 'Peter Drucker'};
    
let userRoles = new Map();
userRoles
    .set(john, 'admin')
    .set(lily, 'editor')
    .set(peter, 'subscriber');
//or
let userRoles = new Map([
    [john, 'admin'],
    [lily, 'editor'],
    [peter, 'subscriber']
]);
userRoles.get(john); // admin
userRoles.get(foo); //undefined
userRoles.has(foo); // false
userRoles.has(lily); // true
console.log(userRoles.size); // 3
for (const user of userRoles.keys()) {}
for (let role of userRoles.values()) {}

//returns an iterator object that contains an array of **[key,value]** of each element in the **Map** object
for (const role of userRoles.entries()) {} 
for (let [user, role] of userRoles.entries()) {}
var keys = [...userRoles.keys()];
userRoles.delete(john);
userRoles.clear();
userRoles.forEach((role, user) => console.log(**${user.name}: ${role}**));
```
- A WeakMap is similar to a Map except for the keys of a WeakMap must be objects
- A WeakMap only has subset methods of a Map object, **get(key)**, **set(key, value)**, **has(key)**, **delete(key)**

# Set
- ES6 provides a new type **Set** that stores a collection of unique values of any type
```javascript
let chars = new Set(['a', 'a', 'b', 'c', 'c']);
console.log(chars); //Set { 'a', 'b', 'c' }
console.log(chars.size);//  3
chars.add('d'); //Set { 'a', 'b', 'c', 'd' }
chars.add('e').add('f'); ////Set { 'a', 'b', 'c', 'd','e','f' }
chars.has('a'); //true
chars.has('z'); //false
chars.delete('f'); //Set { 'a', 'b', 'c', 'd', 'e' }
chars.clear(); // Set{}

let roles = new Set();
roles.add('admin')
    .add('editor')
    .add('subscriber');

for (let role of roles) {}

//The Set also provides the keys(), values(), and entries() methods like the Map. However, keys and values in the Set are identical.

for (let [key, value] of roles.entries()) {
    console.log(key === value);
}
//true, true, true

roles.forEach(role => console.log(role.toUpperCase()));
```
- A WeakSet is similar to a Set except that it contains only objects.
- Since objects in a WeakSet may be automatically garbage-collected, a WeakSet does not have size property.
- Like a WeakMap, you cannot iterate elements of a WeakSet, therefore, you will find that WeakSet is rarely used in practice.

# LET, VAR, CONST
## scop
- var - belongs to global if defined outside a function
- var - local when declare inside a function
- let - block scoped {} (not essentialy function)
- var - global vars added to the global object (window in browser, global in Node.js)
- let - not added to the global object
- var - allow to redeclare
- let - not allow to redeclare
- let - have temporal dead zones 
- var - does not have temporal dead zones
- In the creation phase, assigns storage spaces to var variables and immediately initializes them to undefined.
- In the execution phase, assigns the var variables the values specified by the assignments if there are ones. Otherwise, the var variables remain undefined.
- In the creation phase, assigns storage spaces to the let variables but does not initialize the variables. Referencing uninitialized variables will cause a ReferenceError.
- The let variables have the same execution phase as the var variables.
- const - blocked scoped
- const - can't be reassigned
- The const keyword creates a read-only reference to a value. The readonly reference cannot be reassigned but the value can be changed.

```javascript
//Objects
const person = { age: 20 };
person.age = 30; // OK
console.log(person.age); // 30
person = { age: 40 }; // TypeError

//If you want the value of the person object to be immutable
const person = Object.freeze({age: 20});
person.age = 30; // TypeError

//Note that Object.freeze() is shallow, meaning that it can freeze the properties of the object, not the objects referenced by the properties.

const company = Object.freeze({
    name: 'ABC corp',
    address: {
        street: 'North 1st street',
        city: 'San Jose',
        state: 'CA',
        zipcode: 95134
    }
});
company.address.country = 'USA'; // OK

//Arrays
const colors = ['red'];
colors.push('green');
console.log(colors); // ["red", "green"]

colors.pop();
colors.pop();
console.log(colors); // []

colors = []; // TypeError

//Loops
for (const score of scores) {} //A new **score** constant is created in each iteration.
for (const i = 0; i < scores.length; i++) {} // TypeError //Declaration is only evaluated once before the loop body starts.
```
<div style="page-break-after: always;"></div>

# Advanced Operators
- short-circuited means the operator does not evaluate the second value if the first operand is neither undefined nor null
- Below 4 are short-circuits
- 
## The Logical OR assignment operator (x ||= y) 
- The logical OR assignment operator **only assigns** y to x if x is **falsy**
- Is equivalent to **x || (x = y)**

## The Logical AND assignment operator (x &&= y)
- The logical AND assignment operator **only assigns** y to x if x is **truthy**
- Is equivalent to **x && (x = y)**

## The nullish coalescing assignment operator (x ??= y)
- The nullish coalescing assignment operator **only assigns** y to x **if x is null or undefined**
- Is equivalent to **x ?? (x = y)**

## Nullish Coalescing Operator
- A nullish value is a value that is either null or undefined.
- value1 ?? value2
- The nullish coalescing operator **returns the second value (value2)** if the **first value (value1) is null or undefined**

```javascript
let count = 0;
let result = count || 1; //1
let result2 = count ?? 1; //0
```

## Exponentiation Operator
- x**n, The operator ** raises the x to the power of an exponent n
- An alternative way for **Math.pow(base, exponent)**










