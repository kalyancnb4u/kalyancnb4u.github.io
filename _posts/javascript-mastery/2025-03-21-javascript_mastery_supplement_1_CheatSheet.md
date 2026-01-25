---
title: "JavaScript Mastery - Supplement 1: Quick Reference Cheat Sheet"
date: 2025-03-21 00:00:00 +0530
categories: [JavaScript, JavaScript Mastery]
tags: [JavaScript, Programming, Reference, Cheat Sheet, Quick-guide, Syntax, Patterns]
---

# JavaScript Mastery - Quick Reference Cheat Sheet

## Core Syntax

### Variables
```javascript
const immutable = 'cannot reassign';
let mutable = 'can reassign';
var legacy = 'avoid using';
```

### Data Types
```javascript
// Primitives
String: 'text'
Number: 42, 3.14
Boolean: true, false
Null: null
Undefined: undefined
Symbol: Symbol('id')
BigInt: 9007199254740991n

// Object
Object: {}, []
Function: function() {}
```

### Type Coercion
```javascript
'' == false          // true (loose)
'' === false         // false (strict)
null == undefined    // true
'5' + 3             // '53' (string)
'5' - 3             // 2 (number)
Boolean('')         // false
```

### Operators
```javascript
// Arithmetic
+, -, *, /, %, **

// Assignment
=, +=, -=, *=, /=, %=, **=

// Comparison
==, ===, !=, !==, >, <, >=, <=

// Logical
&&, ||, !

// Nullish coalescing
?? (null/undefined only)

// Optional chaining
obj?.prop?.nested
```

---

## Functions

### Function Types
```javascript
// Declaration
function greet(name) {
  return `Hello ${name}`;
}

// Expression
const greet = function(name) {
  return `Hello ${name}`;
};

// Arrow
const greet = (name) => `Hello ${name}`;
const greet = name => `Hello ${name}`; // Single param
const add = (a, b) => a + b; // Implicit return

// IIFE
(function() {
  console.log('Executed immediately');
})();
```

### Parameters
```javascript
// Default
function greet(name = 'Guest') {}

// Rest
function sum(...numbers) {}

// Destructuring
function process({ name, age }) {}
function getFirst([first, ...rest]) {}
```

---

## Arrays

### Creation & Access
```javascript
const arr = [1, 2, 3];
arr[0]              // 1
arr.length          // 3
arr.at(-1)          // 3 (last element)
```

### Common Methods
```javascript
// Add/Remove
arr.push(4)         // Add to end
arr.pop()           // Remove from end
arr.unshift(0)      // Add to start
arr.shift()         // Remove from start
arr.splice(1, 2)    // Remove 2 items at index 1

// Iteration
arr.forEach(x => console.log(x))
arr.map(x => x * 2)
arr.filter(x => x > 2)
arr.reduce((acc, x) => acc + x, 0)
arr.find(x => x > 2)
arr.findIndex(x => x > 2)
arr.some(x => x > 2)
arr.every(x => x > 0)

// Transformation
arr.slice(1, 3)     // Extract [1,3)
arr.concat([4, 5])  // Combine
arr.join(', ')      // To string
arr.flat()          // Flatten
arr.sort()          // Sort (mutates!)
arr.reverse()       // Reverse (mutates!)
```

---

## Objects

### Creation & Access
```javascript
const obj = { name: 'John', age: 30 };
obj.name            // Dot notation
obj['age']          // Bracket notation
obj?.nested?.prop   // Optional chaining
```

### Common Methods
```javascript
Object.keys(obj)              // ['name', 'age']
Object.values(obj)            // ['John', 30]
Object.entries(obj)           // [['name', 'John'], ...]
Object.assign({}, obj)        // Shallow copy
Object.freeze(obj)            // Immutable
Object.seal(obj)              // No add/delete
Object.hasOwn(obj, 'name')    // Check property
```

### Destructuring
```javascript
const { name, age } = obj;
const { name: userName } = obj; // Rename
const { city = 'NYC' } = obj;   // Default
const { ...rest } = obj;        // Rest
```

---

## Strings

### Template Literals
```javascript
const name = 'John';
`Hello ${name}`               // Interpolation
`Line 1
Line 2`                       // Multiline
```

### Common Methods
```javascript
str.length
str.toUpperCase()
str.toLowerCase()
str.trim()
str.slice(0, 3)
str.substring(0, 3)
str.split(' ')
str.replace('old', 'new')
str.replaceAll('old', 'new')
str.includes('text')
str.startsWith('He')
str.endsWith('lo')
str.padStart(5, '0')
str.repeat(3)
```

---

## Control Flow

### Conditionals
```javascript
// If-else
if (condition) {
} else if (otherCondition) {
} else {
}

// Ternary
condition ? ifTrue : ifFalse

// Switch
switch (value) {
  case 1:
    break;
  default:
}
```

### Loops
```javascript
// For
for (let i = 0; i < 10; i++) {}

// For...of (values)
for (const item of array) {}

// For...in (keys)
for (const key in object) {}

// While
while (condition) {}

// Do-while
do {} while (condition);

// Break/Continue
break;    // Exit loop
continue; // Skip iteration
```

---

## Async Programming

### Promises
```javascript
// Create
const promise = new Promise((resolve, reject) => {
  if (success) resolve(value);
  else reject(error);
});

// Consume
promise
  .then(result => {})
  .catch(error => {})
  .finally(() => {});

// Utilities
Promise.all([p1, p2])       // Wait for all
Promise.race([p1, p2])      // First to complete
Promise.allSettled([p1, p2])// All results
Promise.any([p1, p2])       // First success
```

### Async/Await
```javascript
async function fetchData() {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return data;
  } catch (error) {
    console.error(error);
  }
}
```

---

## Classes

### Class Syntax
```javascript
class Person {
  // Private field
  #privateField = 'private';
  
  // Constructor
  constructor(name) {
    this.name = name;
  }
  
  // Method
  greet() {
    return `Hello ${this.name}`;
  }
  
  // Static method
  static create(name) {
    return new Person(name);
  }
  
  // Getter
  get fullName() {
    return this.name;
  }
  
  // Setter
  set fullName(value) {
    this.name = value;
  }
}

// Inheritance
class Employee extends Person {
  constructor(name, title) {
    super(name);
    this.title = title;
  }
}
```

---

## Modules

### ES Modules
```javascript
// Export
export const value = 42;
export function func() {}
export default MyClass;

// Import
import MyClass from './module';
import { value, func } from './module';
import * as utils from './module';
import { func as myFunc } from './module';
```

### CommonJS
```javascript
// Export
module.exports = value;
exports.func = function() {};

// Import
const value = require('./module');
const { func } = require('./module');
```

---

## DOM Manipulation

### Selection
```javascript
document.getElementById('id')
document.querySelector('.class')
document.querySelectorAll('div')
document.getElementsByClassName('class')
document.getElementsByTagName('div')
```

### Manipulation
```javascript
// Content
element.textContent = 'text';
element.innerHTML = '<p>HTML</p>';

// Attributes
element.getAttribute('href');
element.setAttribute('href', 'url');
element.dataset.customAttr;

// Classes
element.classList.add('class');
element.classList.remove('class');
element.classList.toggle('class');
element.classList.contains('class');

// Styles
element.style.color = 'red';
element.style.display = 'none';

// Creation
const newEl = document.createElement('div');
parent.appendChild(newEl);
parent.removeChild(child);
```

### Events
```javascript
element.addEventListener('click', (e) => {
  e.preventDefault();
  e.stopPropagation();
  console.log(e.target);
});

element.removeEventListener('click', handler);
```

---

## Common Patterns

### Destructuring
```javascript
// Array
const [a, b, ...rest] = [1, 2, 3, 4];

// Object
const { name, age } = person;

// Swap
[a, b] = [b, a];
```

### Spread/Rest
```javascript
// Spread
const arr2 = [...arr1];
const obj2 = {...obj1};
func(...args);

// Rest
function sum(...numbers) {}
const [first, ...rest] = arr;
const {a, ...others} = obj;
```

### Closures
```javascript
function outer() {
  const secret = 'hidden';
  return function inner() {
    return secret;
  };
}
```

### Debounce/Throttle
```javascript
// Debounce
function debounce(fn, delay) {
  let timeout;
  return (...args) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => fn(...args), delay);
  };
}

// Throttle
function throttle(fn, interval) {
  let lastCall = 0;
  return (...args) => {
    const now = Date.now();
    if (now - lastCall >= interval) {
      lastCall = now;
      fn(...args);
    }
  };
}
```

---

## RegEx Quick Reference

```javascript
// Patterns
/pattern/flags
new RegExp('pattern', 'flags')

// Flags
g   // Global
i   // Case-insensitive
m   // Multiline
s   // Dot matches newline
u   // Unicode
y   // Sticky

// Character classes
\d  // Digit [0-9]
\w  // Word [A-Za-z0-9_]
\s  // Whitespace
.   // Any character
[]  // Character set
[^] // Negated set

// Quantifiers
*   // 0 or more
+   // 1 or more
?   // 0 or 1
{n} // Exactly n
{n,}// n or more
{n,m}// n to m

// Anchors
^   // Start of string
$   // End of string
\b  // Word boundary

// Methods
str.match(/pattern/)
str.replace(/pattern/, 'replacement')
str.split(/pattern/)
/pattern/.test(str)
/pattern/.exec(str)
```

---

## Error Handling

```javascript
// Try-catch
try {
  riskyOperation();
} catch (error) {
  console.error(error.message);
} finally {
  cleanup();
}

// Custom error
class CustomError extends Error {
  constructor(message) {
    super(message);
    this.name = 'CustomError';
  }
}

throw new CustomError('Something went wrong');
```

---

## Performance Tips

### Memory
```javascript
// WeakMap/WeakSet for garbage collection
const cache = new WeakMap();

// Clear references
obj = null;

// Clear timers
clearTimeout(id);
clearInterval(id);
```

### Optimization
```javascript
// Memoization
const memo = {};
if (memo[key]) return memo[key];
memo[key] = expensiveOp();

// Lazy loading
const Module = lazy(() => import('./Module'));

// Code splitting
import('./module').then(mod => {});
```

---

## Common Gotchas

```javascript
// == vs ===
5 == '5'    // true
5 === '5'   // false

// this binding
obj.method() // this = obj
method()     // this = window/undefined
() => {}     // this = lexical

// Array/Object comparison
[] === []    // false (different references)
{} === {}    // false

// Hoisting
console.log(x); // undefined (var)
console.log(y); // ReferenceError (let/const)

// Falsy values
false, 0, '', null, undefined, NaN

// Type coercion
'5' - 3      // 2 (number)
'5' + 3      // '53' (string)

// NaN
NaN === NaN  // false
Number.isNaN(NaN) // true
```

---

## Quick Tips

✅ **Always use `const` by default, `let` when needed, avoid `var`**
✅ **Prefer `===` over `==`**
✅ **Use arrow functions for lexical `this`**
✅ **Destructure objects/arrays for cleaner code**
✅ **Use template literals for strings**
✅ **Prefer `async/await` over promise chains**
✅ **Use optional chaining `?.` to avoid errors**
✅ **Use nullish coalescing `??` for defaults**
✅ **Always handle errors with try-catch**
✅ **Use `Array.from()` to convert array-like objects**

---

**Print this cheat sheet for quick reference while coding!** 📝

---

*JavaScript Mastery Series - Quick Reference*
*For complete explanations, see the main series parts*
