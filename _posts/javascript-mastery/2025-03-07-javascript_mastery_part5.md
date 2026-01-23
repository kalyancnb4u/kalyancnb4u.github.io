---
title: "Complete JavaScript Mastery Part 5: Browser APIs and DOM"
date: 2024-01-12 10:00:00 +0000
categories: [JavaScript, Web Development, Browser]
tags: [javascript, dom, browser-apis, events, storage, fetch, web-apis, performance]
---

# Complete JavaScript Mastery Part 5: Browser APIs and DOM

## Introduction

The browser environment provides a rich set of APIs that enable JavaScript to interact with web pages, handle user input, store data, make network requests, and access device capabilities. Mastering these APIs is essential for building modern web applications.

This part explores:
- DOM manipulation and traversal
- Event handling and delegation
- Browser storage (localStorage, IndexedDB, etc.)
- Fetch API and network requests
- Modern browser APIs (Intersection Observer, Web Workers, etc.)

**Prerequisites:**
- Part 1: JavaScript fundamentals
- Part 2: Event loop, async programming
- Part 3: Advanced patterns

**Why Browser APIs Matter:**

- Create dynamic, interactive user interfaces
- Persist data across sessions
- Communicate with servers efficiently
- Access device capabilities
- Build progressive web applications

Let's master browser APIs and DOM manipulation.

---

## 5.1 DOM Manipulation

### DOM Fundamentals

The Document Object Model (DOM) represents the HTML document as a tree of nodes that JavaScript can manipulate.

**DOM Tree Structure:**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Document</title>
  </head>
  <body>
    <div id="app">
      <h1>Hello World</h1>
      <p>This is a paragraph</p>
    </div>
  </body>
</html>
```

```
Document
  └─ html (Element)
      ├─ head (Element)
      │   └─ title (Element)
      │       └─ "Document" (Text)
      └─ body (Element)
          └─ div (Element, id="app")
              ├─ h1 (Element)
              │   └─ "Hello World" (Text)
              └─ p (Element)
                  └─ "This is a paragraph" (Text)
```

#### Node Types

```javascript
// Common node types
console.log(Node.ELEMENT_NODE);                // 1
console.log(Node.TEXT_NODE);                   // 3
console.log(Node.COMMENT_NODE);                // 8
console.log(Node.DOCUMENT_NODE);               // 9
console.log(Node.DOCUMENT_FRAGMENT_NODE);      // 11

// Checking node type
const element = document.getElementById('app');
console.log(element.nodeType);  // 1 (Element)
console.log(element.nodeName);  // "DIV"
console.log(element.nodeValue); // null (elements have null nodeValue)

// Text node
const textNode = element.firstChild;
console.log(textNode.nodeType);  // 3 (Text)
console.log(textNode.nodeValue); // The text content
```

### Element Selection

#### Classic Methods

```javascript
// By ID - fastest, returns single element or null
const app = document.getElementById('app');

// By class name - returns HTMLCollection (live)
const buttons = document.getElementsByClassName('btn');

// By tag name - returns HTMLCollection (live)
const divs = document.getElementsByTagName('div');

// By name attribute - for form elements
const inputs = document.getElementsByName('username');

// Note: HTMLCollection is live (updates automatically)
const divs = document.getElementsByTagName('div');
console.log(divs.length); // 5
document.body.appendChild(document.createElement('div'));
console.log(divs.length); // 6 (automatically updated!)
```

#### Modern Selectors

```javascript
// querySelector - returns first match or null
const firstButton = document.querySelector('.btn');
const app = document.querySelector('#app');
const firstParagraph = document.querySelector('div > p');

// querySelectorAll - returns NodeList (static)
const allButtons = document.querySelectorAll('.btn');
const paragraphs = document.querySelectorAll('p');

// Complex selectors
const links = document.querySelectorAll('a[href^="https"]');
const evenRows = document.querySelectorAll('tr:nth-child(even)');
const notFirst = document.querySelectorAll('.item:not(:first-child)');

// NodeList vs HTMLCollection
const nodeList = document.querySelectorAll('div');
const htmlCollection = document.getElementsByTagName('div');

// NodeList is static (snapshot at query time)
console.log(nodeList.length); // 5
document.body.appendChild(document.createElement('div'));
console.log(nodeList.length); // Still 5

// NodeList has forEach
nodeList.forEach(div => console.log(div));

// HTMLCollection doesn't (must convert)
Array.from(htmlCollection).forEach(div => console.log(div));
```

#### Closest, Matches, Contains

```javascript
// closest() - finds nearest ancestor matching selector
const button = document.querySelector('.btn');
const card = button.closest('.card'); // Finds parent .card
const form = button.closest('form');  // Finds parent form

// matches() - checks if element matches selector
if (element.matches('.active')) {
  console.log('Element is active');
}

// contains() - checks if element contains another
const parent = document.querySelector('.parent');
const child = document.querySelector('.child');
console.log(parent.contains(child)); // true/false
```

### DOM Manipulation Methods

#### Creating Elements

```javascript
// createElement
const div = document.createElement('div');
div.className = 'card';
div.id = 'main-card';

// createTextNode
const text = document.createTextNode('Hello World');

// createDocumentFragment (for batch operations)
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li);
}
// Single reflow/repaint
document.querySelector('ul').appendChild(fragment);

// cloneNode
const original = document.querySelector('.template');
const clone = original.cloneNode(true); // true = deep clone
```

#### Inserting Elements

```javascript
const container = document.querySelector('.container');
const newElement = document.createElement('div');

// appendChild - adds to end
container.appendChild(newElement);

// append - can add multiple nodes and strings (modern)
container.append(newElement, 'Some text', anotherElement);

// prepend - adds to beginning
container.prepend(newElement);

// insertBefore - insert before reference node
const reference = document.querySelector('.reference');
container.insertBefore(newElement, reference);

// insertAdjacentElement/HTML/Text
const target = document.querySelector('.target');

// Positions: 'beforebegin', 'afterbegin', 'beforeend', 'afterend'
target.insertAdjacentElement('beforebegin', newElement);
target.insertAdjacentHTML('afterbegin', '<p>New paragraph</p>');
target.insertAdjacentText('beforeend', 'Some text');

/*
<!-- beforebegin -->
<div class="target">
  <!-- afterbegin -->
  content
  <!-- beforeend -->
</div>
<!-- afterend -->
*/
```

#### Removing Elements

```javascript
// remove() - removes element from DOM
const element = document.querySelector('.remove-me');
element.remove();

// removeChild - parent removes child
const parent = document.querySelector('.parent');
const child = document.querySelector('.child');
parent.removeChild(child);

// Clear all children
parent.innerHTML = ''; // ❌ Not recommended (event listeners leak)
parent.textContent = ''; // ❌ Not recommended

// ✅ Better: remove children properly
while (parent.firstChild) {
  parent.removeChild(parent.firstChild);
}

// ✅ Modern: replaceChildren
parent.replaceChildren(); // Removes all children
parent.replaceChildren(newChild1, newChild2); // Replace with new children
```

#### Replacing Elements

```javascript
// replaceWith - replaces element
const oldElement = document.querySelector('.old');
const newElement = document.createElement('div');
newElement.textContent = 'New content';
oldElement.replaceWith(newElement);

// replaceChild - parent replaces child
parent.replaceChild(newChild, oldChild);
```

### Element Properties and Attributes

#### Attributes vs Properties

```javascript
const input = document.querySelector('input');

// Attributes (HTML attributes)
input.setAttribute('type', 'text');
input.getAttribute('type'); // "text"
input.hasAttribute('type'); // true
input.removeAttribute('type');

// Properties (DOM properties)
input.type = 'email';
input.value = 'user@example.com';

// Key difference: attributes vs properties
input.setAttribute('value', 'default'); // Sets attribute
console.log(input.getAttribute('value')); // "default"
console.log(input.value); // What user typed (property)

// Boolean attributes
input.setAttribute('disabled', ''); // Any value works
input.disabled = true; // Property is boolean

// Custom attributes (use data-*)
input.setAttribute('data-user-id', '123');
console.log(input.getAttribute('data-user-id')); // "123"
```

#### Dataset API

```javascript
// HTML: <div data-user-id="123" data-user-name="John" data-premium="true">

const element = document.querySelector('div');

// Read data attributes
console.log(element.dataset.userId);   // "123"
console.log(element.dataset.userName); // "John"
console.log(element.dataset.premium);  // "true" (string!)

// Write data attributes
element.dataset.score = '100';
element.dataset.lastLogin = new Date().toISOString();

// Converts camelCase to kebab-case
element.dataset.maxRetries = '3';
// Creates: data-max-retries="3"

// Delete data attribute
delete element.dataset.premium;

// Note: all values are strings
const isPremium = element.dataset.premium === 'true';
const userId = parseInt(element.dataset.userId);
```

#### ClassList API

```javascript
const element = document.querySelector('.element');

// add - add one or more classes
element.classList.add('active');
element.classList.add('highlight', 'selected');

// remove - remove one or more classes
element.classList.remove('active');
element.classList.remove('highlight', 'selected');

// toggle - add if absent, remove if present
element.classList.toggle('active');
element.classList.toggle('active', true);  // Force add
element.classList.toggle('active', false); // Force remove

// contains - check if class exists
if (element.classList.contains('active')) {
  console.log('Element is active');
}

// replace - replace one class with another
element.classList.replace('old-class', 'new-class');

// ❌ BAD: String manipulation
element.className += ' active'; // Can create duplicates
element.className = element.className.replace('active', '');

// ✅ GOOD: classList API
element.classList.add('active');
element.classList.remove('active');
```

#### Style Manipulation

```javascript
const element = document.querySelector('.element');

// Inline styles (camelCase)
element.style.color = 'red';
element.style.backgroundColor = 'blue';
element.style.fontSize = '16px';
element.style.margin = '10px 20px';

// Read computed styles
const computed = window.getComputedStyle(element);
console.log(computed.color);           // "rgb(255, 0, 0)"
console.log(computed.backgroundColor); // "rgb(0, 0, 255)"
console.log(computed.width);           // "500px"

// Read inline vs computed
element.style.color = 'red';
console.log(element.style.color);              // "red"
console.log(getComputedStyle(element).color);  // "rgb(255, 0, 0)"

// Remove inline style
element.style.color = '';
element.style.removeProperty('background-color');

// Set multiple styles
Object.assign(element.style, {
  color: 'white',
  backgroundColor: 'black',
  padding: '10px'
});

// ❌ BAD: Too many inline styles
element.style.display = 'flex';
element.style.justifyContent = 'center';
element.style.alignItems = 'center';

// ✅ GOOD: Use classes
element.classList.add('flex-center');
```

#### Content Manipulation

```javascript
// innerHTML - parses HTML (⚠️ XSS risk with user input)
element.innerHTML = '<p>New <strong>content</strong></p>';
console.log(element.innerHTML); // Includes HTML tags

// textContent - plain text only
element.textContent = 'Plain text content';
console.log(element.textContent); // Just the text

// innerText - respects CSS visibility
element.innerText = 'Visible text';

// Key differences
const div = document.createElement('div');
div.innerHTML = '<span style="display:none">Hidden</span>Visible';
console.log(div.innerHTML);    // "<span...>Hidden</span>Visible"
console.log(div.textContent);  // "HiddenVisible"
console.log(div.innerText);    // "Visible"

// ⚠️ Security: Never use innerHTML with user input
// ❌ DANGEROUS
element.innerHTML = userInput; // XSS vulnerability!

// ✅ SAFE
element.textContent = userInput;

// ✅ SAFE: Sanitize first
element.innerHTML = DOMPurify.sanitize(userInput);
```

### DOM Traversal

```javascript
const element = document.querySelector('.element');

// Parent
element.parentNode;        // Parent node (any type)
element.parentElement;     // Parent element (Element only)

// Children
element.childNodes;        // NodeList (includes text nodes)
element.children;          // HTMLCollection (elements only)
element.firstChild;        // First node (might be text)
element.firstElementChild; // First element
element.lastChild;         // Last node
element.lastElementChild;  // Last element

// Siblings
element.previousSibling;        // Previous node
element.previousElementSibling; // Previous element
element.nextSibling;            // Next node
element.nextElementSibling;     // Next element

// Example: traversing
const list = document.querySelector('ul');

// Get all list items (ignoring text nodes)
const items = Array.from(list.children);

// Get first and last items
const first = list.firstElementChild;
const last = list.lastElementChild;

// Navigate siblings
let current = first;
while (current) {
  console.log(current.textContent);
  current = current.nextElementSibling;
}
```

### Element Dimensions and Position

```javascript
const element = document.querySelector('.element');

// Offset dimensions (includes border)
console.log(element.offsetWidth);  // Total width
console.log(element.offsetHeight); // Total height
console.log(element.offsetLeft);   // Left position relative to offsetParent
console.log(element.offsetTop);    // Top position relative to offsetParent
console.log(element.offsetParent); // Positioned ancestor

// Client dimensions (excludes scrollbar and border)
console.log(element.clientWidth);  // Width - border - scrollbar
console.log(element.clientHeight); // Height - border - scrollbar
console.log(element.clientLeft);   // Left border width
console.log(element.clientTop);    // Top border width

// Scroll dimensions
console.log(element.scrollWidth);  // Total scrollable width
console.log(element.scrollHeight); // Total scrollable height
console.log(element.scrollLeft);   // Horizontal scroll position
console.log(element.scrollTop);    // Vertical scroll position

// getBoundingClientRect - position relative to viewport
const rect = element.getBoundingClientRect();
console.log(rect.top);    // Distance from viewport top
console.log(rect.left);   // Distance from viewport left
console.log(rect.bottom); // Distance from viewport top to element bottom
console.log(rect.right);  // Distance from viewport left to element right
console.log(rect.width);  // Element width
console.log(rect.height); // Element height

// Scrolling
element.scrollTo(0, 100);           // Scroll to position
element.scrollBy(0, 50);            // Scroll by offset
element.scrollIntoView();           // Scroll into view
element.scrollIntoView({            // Smooth scroll
  behavior: 'smooth',
  block: 'center'
});
```

### DOM Performance Optimization

#### Batch DOM Updates

```javascript
// ❌ BAD: Multiple reflows
for (let i = 0; i < 100; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  list.appendChild(li); // Reflow on each append!
}

// ✅ GOOD: Single reflow with DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li);
}
list.appendChild(fragment); // Single reflow

// ✅ GOOD: Build HTML string
const html = Array.from({ length: 100 }, (_, i) => 
  `<li>Item ${i}</li>`
).join('');
list.innerHTML = html;
```

#### Avoid Layout Thrashing

```javascript
// ❌ BAD: Read-write cycle causes layout thrashing
elements.forEach(element => {
  const height = element.offsetHeight; // Read (forces layout)
  element.style.height = height * 2 + 'px'; // Write
});

// ✅ GOOD: Batch reads, then batch writes
const heights = elements.map(element => element.offsetHeight);
elements.forEach((element, i) => {
  element.style.height = heights[i] * 2 + 'px';
});

// ✅ GOOD: Use requestAnimationFrame
function batchUpdate() {
  requestAnimationFrame(() => {
    elements.forEach(element => {
      element.style.transform = 'translateX(100px)';
    });
  });
}
```

#### Minimize Reflows and Repaints

```javascript
// Reflow triggers (expensive):
// - offsetWidth, offsetHeight, clientWidth, clientHeight
// - scrollWidth, scrollHeight, scrollTop, scrollLeft
// - getComputedStyle()
// - Adding/removing elements
// - Changing dimensions or position

// Repaint triggers (less expensive):
// - Changing color, background, visibility
// - No layout changes

// ✅ Optimize: Detach from DOM during updates
const container = document.querySelector('.container');
const parent = container.parentNode;

parent.removeChild(container); // Detach

// Make many changes
for (let i = 0; i < 1000; i++) {
  // Modify container
}

parent.appendChild(container); // Reattach (single reflow)

// ✅ Optimize: Use CSS classes instead of inline styles
element.style.width = '100px';
element.style.height = '100px';
element.style.backgroundColor = 'red';

// Better:
element.className = 'box-style';

// ✅ Optimize: Use transform and opacity (no reflow)
// ❌ Triggers reflow
element.style.left = '100px';
element.style.top = '100px';

// ✅ Uses compositor (no reflow)
element.style.transform = 'translate(100px, 100px)';
```

### Frequently Asked Questions

**Q1: What's the difference between innerHTML, textContent, and innerText?**

**A:** 
- `innerHTML`: Parses HTML, returns/sets HTML string (XSS risk)
- `textContent`: Plain text, includes hidden elements
- `innerText`: Respects CSS visibility, triggers reflow

```javascript
element.innerHTML = '<b>Bold</b>';    // Parses HTML
element.textContent = '<b>Bold</b>';  // Shows literally
element.innerText = 'Text';           // Visible text only
```

**Related Concepts:** DOM manipulation, XSS security, performance

---

**Q2: When should I use querySelector vs getElementById?**

**A:** `getElementById` is faster for single ID lookups. `querySelector` is more flexible but slightly slower. Use `getElementById` for performance-critical code, `querySelector` for complex selectors.

```javascript
// Faster
document.getElementById('app');

// More flexible
document.querySelector('#app .container > .item:first-child');
```

**Related Concepts:** Element selection, performance optimization

---

**Q3: How do I avoid memory leaks with DOM manipulation?**

**A:** Remove event listeners before removing elements, use weak references where appropriate, and clear innerHTML properly.

```javascript
// ❌ Memory leak
element.addEventListener('click', handler);
element.remove(); // Listener still in memory!

// ✅ Clean removal
element.removeEventListener('click', handler);
element.remove();

// ✅ Or use event delegation
```

**Related Concepts:** Memory management, event listeners, garbage collection

---

**Q4: What's the difference between NodeList and HTMLCollection?**

**A:** NodeList can be static or live, has forEach. HTMLCollection is always live, no forEach. Both are array-like but not arrays.

```javascript
// NodeList (static from querySelectorAll)
const nodes = document.querySelectorAll('div');
nodes.forEach(node => console.log(node));

// HTMLCollection (live)
const collection = document.getElementsByTagName('div');
// collection.forEach() // Error! No forEach
Array.from(collection).forEach(node => console.log(node));
```

**Related Concepts:** Element selection, iteration, live vs static collections

---

**Q5: How do I optimize DOM manipulation for performance?**

**A:** Batch updates, use DocumentFragment, minimize reflows, use CSS classes over inline styles, and leverage requestAnimationFrame.

```javascript
// ✅ Batch with DocumentFragment
const fragment = document.createDocumentFragment();
items.forEach(item => fragment.appendChild(createItem(item)));
container.appendChild(fragment); // Single reflow

// ✅ Use CSS classes
element.classList.add('active'); // Better than inline styles

// ✅ Use requestAnimationFrame for animations
requestAnimationFrame(() => {
  element.style.transform = 'translateX(100px)';
});
```

**Related Concepts:** Performance optimization, reflow, repaint, layout thrashing

---

## 5.2 Event Handling

### Event Fundamentals

Events enable JavaScript to respond to user interactions and browser actions.

#### Adding Event Listeners

```javascript
const button = document.querySelector('button');

// addEventListener (modern, preferred)
button.addEventListener('click', function(event) {
  console.log('Button clicked!', event);
});

// Arrow function
button.addEventListener('click', (event) => {
  console.log('Clicked!', event);
});

// Named function (for removal)
function handleClick(event) {
  console.log('Clicked!', event);
}
button.addEventListener('click', handleClick);

// Remove listener
button.removeEventListener('click', handleClick);

// ❌ BAD: Inline handler (limits to one handler)
button.onclick = function() {
  console.log('Clicked');
};

// ❌ BAD: HTML attribute
// <button onclick="handleClick()">Click</button>
```

#### Event Object

```javascript
element.addEventListener('click', function(event) {
  // Event properties
  console.log(event.type);           // "click"
  console.log(event.target);         // Element that triggered event
  console.log(event.currentTarget);  // Element listener is attached to
  console.log(event.timeStamp);      // Time event occurred
  console.log(event.bubbles);        // Does event bubble?
  console.log(event.cancelable);     // Can be cancelled?
  
  // Mouse events
  console.log(event.clientX);        // X relative to viewport
  console.log(event.clientY);        // Y relative to viewport
  console.log(event.pageX);          // X relative to document
  console.log(event.pageY);          // Y relative to document
  console.log(event.offsetX);        // X relative to target
  console.log(event.offsetY);        // Y relative to target
  
  // Keyboard events
  console.log(event.key);            // Key value ("a", "Enter")
  console.log(event.code);           // Physical key ("KeyA", "Enter")
  console.log(event.shiftKey);       // Shift key pressed?
  console.log(event.ctrlKey);        // Ctrl key pressed?
  console.log(event.altKey);         // Alt key pressed?
  console.log(event.metaKey);        // Meta/Cmd key pressed?
  
  // Methods
  event.preventDefault();            // Prevent default action
  event.stopPropagation();           // Stop bubbling
  event.stopImmediatePropagation();  // Stop all handlers
});
```

### Event Propagation

Events propagate through the DOM in three phases: capture → target → bubble.

```javascript
/*
  <div class="outer">
    <div class="inner">
      <button>Click</button>
    </div>
  </div>
*/

// Capture phase (outer → inner → button)
outer.addEventListener('click', () => {
  console.log('Outer (capture)');
}, true); // true = capture phase

inner.addEventListener('click', () => {
  console.log('Inner (capture)');
}, true);

// Target phase (button)
button.addEventListener('click', () => {
  console.log('Button (target)');
});

// Bubble phase (button → inner → outer)
inner.addEventListener('click', () => {
  console.log('Inner (bubble)');
});

outer.addEventListener('click', () => {
  console.log('Outer (bubble)');
});

// Click button logs:
// Outer (capture)
// Inner (capture)
// Button (target)
// Inner (bubble)
// Outer (bubble)
```

#### stopPropagation vs stopImmediatePropagation

```javascript
// stopPropagation - stops bubbling to parent elements
button.addEventListener('click', (e) => {
  e.stopPropagation();
  console.log('Button clicked');
});

outer.addEventListener('click', () => {
  console.log('Outer clicked'); // Won't fire
});

// stopImmediatePropagation - stops all handlers on current element
button.addEventListener('click', (e) => {
  e.stopImmediatePropagation();
  console.log('First handler');
});

button.addEventListener('click', () => {
  console.log('Second handler'); // Won't fire
});
```

### Event Delegation

Handle events on parent instead of individual children (efficient for many elements).

```javascript
// ❌ BAD: Add listener to each item
const items = document.querySelectorAll('.item');
items.forEach(item => {
  item.addEventListener('click', handleClick);
});

// ✅ GOOD: Single listener on parent
const list = document.querySelector('.list');
list.addEventListener('click', (event) => {
  // Check if clicked element is an item
  if (event.target.matches('.item')) {
    handleClick(event);
  }
});

// Works with dynamically added items
const newItem = document.createElement('div');
newItem.className = 'item';
list.appendChild(newItem); // Automatically handled!

// More complex delegation
list.addEventListener('click', (event) => {
  // Handle different elements
  if (event.target.matches('.delete-btn')) {
    deleteItem(event);
  } else if (event.target.matches('.edit-btn')) {
    editItem(event);
  } else if (event.target.matches('.item')) {
    selectItem(event);
  }
});

// Using closest() for nested elements
list.addEventListener('click', (event) => {
  const item = event.target.closest('.item');
  if (item && list.contains(item)) {
    console.log('Item clicked:', item);
  }
});
```

### Common Event Types

#### Mouse Events

```javascript
element.addEventListener('click', handler);        // Click
element.addEventListener('dblclick', handler);     // Double click
element.addEventListener('mousedown', handler);    // Mouse button pressed
element.addEventListener('mouseup', handler);      // Mouse button released
element.addEventListener('mousemove', handler);    // Mouse moved
element.addEventListener('mouseenter', handler);   // Mouse enters (no bubble)
element.addEventListener('mouseleave', handler);   // Mouse leaves (no bubble)
element.addEventListener('mouseover', handler);    // Mouse enters (bubbles)
element.addEventListener('mouseout', handler);     // Mouse leaves (bubbles)
element.addEventListener('contextmenu', handler);  // Right click

// Drag events
element.addEventListener('dragstart', handler);
element.addEventListener('drag', handler);
element.addEventListener('dragend', handler);
element.addEventListener('dragenter', handler);
element.addEventListener('dragover', handler);
element.addEventListener('dragleave', handler);
element.addEventListener('drop', handler);
```

#### Keyboard Events

```javascript
// Keyboard events fire on focused element or document
document.addEventListener('keydown', (event) => {
  console.log('Key pressed:', event.key);
  console.log('Key code:', event.code);
  
  // Check for specific keys
  if (event.key === 'Enter') {
    console.log('Enter pressed');
  }
  
  if (event.key === 'Escape') {
    console.log('Escape pressed');
  }
  
  // Check for modifiers
  if (event.ctrlKey && event.key === 's') {
    event.preventDefault(); // Prevent browser save
    console.log('Ctrl+S pressed');
  }
});

document.addEventListener('keyup', handler);    // Key released
document.addEventListener('keypress', handler); // Deprecated

// Key combinations
function handleKeyCombo(event) {
  const combo = [
    event.ctrlKey && 'Ctrl',
    event.shiftKey && 'Shift',
    event.altKey && 'Alt',
    event.metaKey && 'Meta',
    event.key
  ].filter(Boolean).join('+');
  
  console.log('Combo:', combo);
  // Example: "Ctrl+Shift+A"
}
```

#### Form Events

```javascript
const form = document.querySelector('form');
const input = document.querySelector('input');

// Form submission
form.addEventListener('submit', (event) => {
  event.preventDefault(); // Prevent page reload
  
  const formData = new FormData(form);
  console.log(Object.fromEntries(formData));
});

// Input events
input.addEventListener('input', (event) => {
  console.log('Value:', event.target.value); // Fires on every change
});

input.addEventListener('change', (event) => {
  console.log('Changed:', event.target.value); // Fires on blur
});

input.addEventListener('focus', () => {
  console.log('Input focused');
});

input.addEventListener('blur', () => {
  console.log('Input blurred');
});

// Form validation
input.addEventListener('invalid', (event) => {
  console.log('Invalid input');
  event.preventDefault(); // Prevent browser validation UI
});
```

#### Window Events

```javascript
// Load events
window.addEventListener('load', () => {
  console.log('Page fully loaded');
});

window.addEventListener('DOMContentLoaded', () => {
  console.log('DOM ready (before images)');
});

// Resize and scroll
window.addEventListener('resize', () => {
  console.log('Window resized:', window.innerWidth, window.innerHeight);
});

window.addEventListener('scroll', () => {
  console.log('Scrolled:', window.scrollY);
});

// Before unload
window.addEventListener('beforeunload', (event) => {
  // Show confirmation dialog
  event.preventDefault();
  event.returnValue = ''; // Required for Chrome
});

// Visibility change
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    console.log('Tab hidden');
  } else {
    console.log('Tab visible');
  }
});
```

### Custom Events

Create and dispatch custom events for component communication.

```javascript
// Create custom event
const event = new CustomEvent('userLoggedIn', {
  detail: {
    userId: 123,
    username: 'john',
    timestamp: Date.now()
  },
  bubbles: true,
  cancelable: true
});

// Dispatch event
element.dispatchEvent(event);

// Listen for custom event
document.addEventListener('userLoggedIn', (event) => {
  console.log('User logged in:', event.detail);
});

// Practical example: Component communication
class EventBus {
  constructor() {
    this.element = document.createElement('div');
  }
  
  on(event, callback) {
    this.element.addEventListener(event, callback);
  }
  
  off(event, callback) {
    this.element.removeEventListener(event, callback);
  }
  
  emit(event, data) {
    this.element.dispatchEvent(new CustomEvent(event, {
      detail: data
    }));
  }
}

// Usage
const bus = new EventBus();

bus.on('dataUpdated', (event) => {
  console.log('Data updated:', event.detail);
});

bus.emit('dataUpdated', { items: [1, 2, 3] });
```

### Event Performance

#### Debouncing and Throttling

```javascript
// Debounce - wait for pause in events
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Usage
const searchInput = document.querySelector('#search');
const debouncedSearch = debounce((event) => {
  console.log('Searching:', event.target.value);
  // Make API call
}, 300);

searchInput.addEventListener('input', debouncedSearch);

// Throttle - limit frequency
function throttle(fn, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}

// Usage
const throttledScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 100);

window.addEventListener('scroll', throttledScroll);
```

#### Passive Event Listeners

```javascript
// Passive listeners improve scroll performance
// Indicates you won't call preventDefault()

// ❌ Non-passive (blocks scroll until handler completes)
element.addEventListener('touchstart', handler);

// ✅ Passive (doesn't block scroll)
element.addEventListener('touchstart', handler, { passive: true });

// Multiple options
element.addEventListener('scroll', handler, {
  passive: true,
  capture: false,
  once: false
});
```

### Interview Questions

**Question 1: What's the difference between event.target and event.currentTarget?**

**Difficulty:** Junior

**Answer:**
- `event.target`: Element that triggered the event (clicked element)
- `event.currentTarget`: Element the listener is attached to

```javascript
div.addEventListener('click', (event) => {
  console.log(event.target);        // <button> (what you clicked)
  console.log(event.currentTarget); // <div> (where listener is)
});
```

**Follow-up:**
* When would they be the same?
* How does this relate to event delegation?

---

**Question 2: Implement event delegation for a dynamic list.**

**Difficulty:** Mid-Level

**Answer:**

```javascript
const list = document.querySelector('#list');

list.addEventListener('click', (event) => {
  // Find clicked item (even if clicked child element)
  const item = event.target.closest('.list-item');
  
  if (item && list.contains(item)) {
    // Check which button was clicked
    if (event.target.matches('.delete-btn')) {
      item.remove();
    } else if (event.target.matches('.edit-btn')) {
      const text = prompt('New text:');
      if (text) item.textContent = text;
    }
  }
});

// Add new items dynamically
function addItem(text) {
  const item = document.createElement('div');
  item.className = 'list-item';
  item.innerHTML = `
    ${text}
    <button class="edit-btn">Edit</button>
    <button class="delete-btn">Delete</button>
  `;
  list.appendChild(item);
}
```

**Why This Matters:** Event delegation is crucial for performance with large lists and dynamic content.

---

**Question 3: Explain event bubbling and provide an example of when you'd stop it.**

**Difficulty:** Mid-Level

**Answer:**

Event bubbling means events propagate from target element up through ancestors. Stop it when you want to prevent parent handlers from firing.

```javascript
// Bubbling example
outer.addEventListener('click', () => {
  console.log('Outer');
});

inner.addEventListener('click', (e) => {
  console.log('Inner');
  e.stopPropagation(); // Stops outer handler
});

// Use case: Modal with backdrop
backdrop.addEventListener('click', () => {
  closeModal();
});

modal.addEventListener('click', (e) => {
  e.stopPropagation(); // Clicking modal content doesn't close
});
```

**Follow-up:**
* What's the difference between stopPropagation and stopImmediatePropagation?
* What's event capturing?

---

### Key Takeaways

✅ **DOM Manipulation:**
- Use modern selection methods (querySelector)
- Batch DOM updates for performance
- Prefer classList over className
- Use textContent for security
- DocumentFragment for bulk inserts

✅ **Event Handling:**
- addEventListener is preferred over inline handlers
- Event delegation for dynamic content
- Understand bubbling and capturing
- Use preventDefault() to stop default actions
- Debounce/throttle expensive handlers

✅ **Performance:**
- Minimize reflows and repaints
- Use event delegation
- Passive event listeners for scroll
- RequestAnimationFrame for animations
- Batch reads and writes

---

## 5.3 Browser Storage

Modern browsers provide multiple storage mechanisms for persisting data on the client side.

### Storage Comparison

| Feature | Cookies | localStorage | sessionStorage | IndexedDB | Cache API |
|---------|---------|--------------|----------------|-----------|-----------|
| **Capacity** | 4KB | 5-10MB | 5-10MB | 50MB+ | Limited by quota |
| **Persistence** | Expirable | Permanent | Session | Permanent | Permanent |
| **Scope** | Domain | Domain | Tab | Domain | Domain |
| **Sent to Server** | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **API** | String | Sync | Sync | Async | Async |
| **Use Case** | Auth tokens | Preferences | Temp data | Large data | Offline |

### Cookies

Cookies are small text strings sent with every HTTP request.

```javascript
// Setting cookies
document.cookie = "username=john";
document.cookie = "sessionId=abc123";

// With options
document.cookie = "token=xyz; max-age=3600; path=/; secure; samesite=strict";

// Cookie options
// - max-age: Lifetime in seconds
// - expires: Expiration date (Date.toUTCString())
// - path: URL path (/admin, / for all)
// - domain: Domain (.example.com for subdomains)
// - secure: HTTPS only
// - httpOnly: No JavaScript access (server-side only)
// - samesite: CSRF protection (strict, lax, none)

// Reading cookies
console.log(document.cookie); // "username=john; sessionId=abc123"

// Parse cookies
function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) {
    return parts.pop().split(';').shift();
  }
}

console.log(getCookie('username')); // "john"

// Delete cookie (set max-age to 0)
document.cookie = "username=; max-age=0";

// Cookie utility class
class CookieManager {
  static set(name, value, options = {}) {
    let cookie = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`;
    
    if (options.maxAge) {
      cookie += `; max-age=${options.maxAge}`;
    }
    
    if (options.expires) {
      cookie += `; expires=${options.expires.toUTCString()}`;
    }
    
    if (options.path) {
      cookie += `; path=${options.path}`;
    }
    
    if (options.domain) {
      cookie += `; domain=${options.domain}`;
    }
    
    if (options.secure) {
      cookie += '; secure';
    }
    
    if (options.sameSite) {
      cookie += `; samesite=${options.sameSite}`;
    }
    
    document.cookie = cookie;
  }
  
  static get(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) {
      return decodeURIComponent(parts.pop().split(';').shift());
    }
    return null;
  }
  
  static delete(name, options = {}) {
    this.set(name, '', { ...options, maxAge: 0 });
  }
  
  static getAll() {
    return document.cookie.split('; ').reduce((acc, cookie) => {
      const [name, value] = cookie.split('=');
      acc[decodeURIComponent(name)] = decodeURIComponent(value);
      return acc;
    }, {});
  }
}

// Usage
CookieManager.set('user', 'john', {
  maxAge: 3600,
  path: '/',
  secure: true,
  sameSite: 'strict'
});

console.log(CookieManager.get('user')); // "john"
console.log(CookieManager.getAll()); // { user: 'john', ... }
CookieManager.delete('user');
```

**Cookie Security Best Practices:**

```javascript
// ✅ GOOD: Secure cookie settings
CookieManager.set('authToken', token, {
  maxAge: 3600,
  path: '/',
  secure: true,        // HTTPS only
  sameSite: 'strict',  // CSRF protection
  // httpOnly: true    // Set server-side only
});

// ❌ BAD: Insecure cookies
document.cookie = "authToken=abc123"; // No security flags!

// Third-party cookies (being phased out)
document.cookie = "tracking=xyz; domain=.example.com; sameSite=none; secure";
```

### Web Storage API

localStorage and sessionStorage provide simple key-value storage.

#### localStorage

```javascript
// Set item (converts to string)
localStorage.setItem('username', 'john');
localStorage.setItem('count', 42);

// Get item (returns string or null)
const username = localStorage.getItem('username'); // "john"
const count = localStorage.getItem('count');       // "42"

// Remove item
localStorage.removeItem('username');

// Clear all
localStorage.clear();

// Check existence
if (localStorage.getItem('theme') === null) {
  localStorage.setItem('theme', 'dark');
}

// Object storage (must stringify)
const user = { name: 'John', age: 30 };
localStorage.setItem('user', JSON.stringify(user));

const stored = JSON.parse(localStorage.getItem('user'));
console.log(stored.name); // "John"

// Storage length
console.log(localStorage.length);

// Iterate over items
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  console.log(key, localStorage.getItem(key));
}

// Better iteration
Object.keys(localStorage).forEach(key => {
  console.log(key, localStorage.getItem(key));
});
```

#### sessionStorage

Same API as localStorage, but data persists only for the session (tab/window).

```javascript
// Tab 1
sessionStorage.setItem('tabId', '1');

// Tab 2
console.log(sessionStorage.getItem('tabId')); // null (different session)

// Cleared when tab/window closes
sessionStorage.setItem('tempData', 'value');
// Close tab → data lost
```

#### Storage Events

Listen for storage changes from other tabs/windows:

```javascript
// Tab 1
window.addEventListener('storage', (event) => {
  console.log('Storage changed:');
  console.log('Key:', event.key);
  console.log('Old value:', event.oldValue);
  console.log('New value:', event.newValue);
  console.log('URL:', event.url);
  console.log('Storage:', event.storageArea);
});

// Tab 2
localStorage.setItem('theme', 'dark');
// Tab 1 receives storage event

// Note: Event only fires in OTHER tabs, not the tab that made the change
```

#### Storage Wrapper with Type Safety

```javascript
class TypedStorage {
  constructor(storage = localStorage) {
    this.storage = storage;
  }
  
  set(key, value) {
    try {
      this.storage.setItem(key, JSON.stringify(value));
      return true;
    } catch (error) {
      console.error('Storage error:', error);
      return false;
    }
  }
  
  get(key, defaultValue = null) {
    try {
      const item = this.storage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      console.error('Parse error:', error);
      return defaultValue;
    }
  }
  
  remove(key) {
    this.storage.removeItem(key);
  }
  
  clear() {
    this.storage.clear();
  }
  
  has(key) {
    return this.storage.getItem(key) !== null;
  }
  
  keys() {
    return Object.keys(this.storage);
  }
  
  size() {
    return this.storage.length;
  }
  
  // With expiration
  setWithExpiry(key, value, ttl) {
    const item = {
      value,
      expiry: Date.now() + ttl
    };
    this.set(key, item);
  }
  
  getWithExpiry(key) {
    const item = this.get(key);
    if (!item) return null;
    
    if (Date.now() > item.expiry) {
      this.remove(key);
      return null;
    }
    
    return item.value;
  }
}

// Usage
const storage = new TypedStorage(localStorage);

// Store objects directly
storage.set('user', { name: 'John', age: 30 });
const user = storage.get('user'); // Already parsed

// With expiration (5 minutes)
storage.setWithExpiry('token', 'abc123', 5 * 60 * 1000);
const token = storage.getWithExpiry('token');
```

### IndexedDB

IndexedDB is a low-level API for storing large amounts of structured data.

#### Basic IndexedDB Operations

```javascript
// Open database
const request = indexedDB.open('MyDatabase', 1);

// Database upgrade (create stores)
request.onupgradeneeded = (event) => {
  const db = event.target.result;
  
  // Create object store (table)
  if (!db.objectStoreNames.contains('users')) {
    const store = db.createObjectStore('users', {
      keyPath: 'id',
      autoIncrement: true
    });
    
    // Create indexes
    store.createIndex('email', 'email', { unique: true });
    store.createIndex('age', 'age', { unique: false });
  }
};

// Success
request.onsuccess = (event) => {
  const db = event.target.result;
  console.log('Database opened:', db);
};

// Error
request.onerror = (event) => {
  console.error('Database error:', event.target.error);
};

// Add data
function addUser(db, user) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  const request = store.add(user);
  
  request.onsuccess = () => {
    console.log('User added:', request.result);
  };
  
  request.onerror = () => {
    console.error('Add error:', request.error);
  };
}

// Get data
function getUser(db, id) {
  return new Promise((resolve, reject) => {
    const transaction = db.transaction(['users'], 'readonly');
    const store = transaction.objectStore('users');
    const request = store.get(id);
    
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}

// Get all data
function getAllUsers(db) {
  return new Promise((resolve, reject) => {
    const transaction = db.transaction(['users'], 'readonly');
    const store = transaction.objectStore('users');
    const request = store.getAll();
    
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}

// Update data
function updateUser(db, user) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  const request = store.put(user);
  
  request.onsuccess = () => {
    console.log('User updated');
  };
}

// Delete data
function deleteUser(db, id) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  const request = store.delete(id);
  
  request.onsuccess = () => {
    console.log('User deleted');
  };
}

// Query by index
function getUserByEmail(db, email) {
  return new Promise((resolve, reject) => {
    const transaction = db.transaction(['users'], 'readonly');
    const store = transaction.objectStore('users');
    const index = store.index('email');
    const request = index.get(email);
    
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}
```

#### IndexedDB Wrapper

```javascript
class IndexedDBWrapper {
  constructor(dbName, version = 1) {
    this.dbName = dbName;
    this.version = version;
    this.db = null;
  }
  
  async open(stores) {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        
        stores.forEach(({ name, options, indexes }) => {
          if (!db.objectStoreNames.contains(name)) {
            const store = db.createObjectStore(name, options);
            
            if (indexes) {
              indexes.forEach(({ name, keyPath, options }) => {
                store.createIndex(name, keyPath, options);
              });
            }
          }
        });
      };
      
      request.onsuccess = () => {
        this.db = request.result;
        resolve(this.db);
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  async add(storeName, data) {
    return this._execute(storeName, 'readwrite', (store) => store.add(data));
  }
  
  async get(storeName, key) {
    return this._execute(storeName, 'readonly', (store) => store.get(key));
  }
  
  async getAll(storeName) {
    return this._execute(storeName, 'readonly', (store) => store.getAll());
  }
  
  async update(storeName, data) {
    return this._execute(storeName, 'readwrite', (store) => store.put(data));
  }
  
  async delete(storeName, key) {
    return this._execute(storeName, 'readwrite', (store) => store.delete(key));
  }
  
  async clear(storeName) {
    return this._execute(storeName, 'readwrite', (store) => store.clear());
  }
  
  async _execute(storeName, mode, operation) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([storeName], mode);
      const store = transaction.objectStore(storeName);
      const request = operation(store);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
}

// Usage
const db = new IndexedDBWrapper('MyApp', 1);

await db.open([
  {
    name: 'users',
    options: { keyPath: 'id', autoIncrement: true },
    indexes: [
      { name: 'email', keyPath: 'email', options: { unique: true } }
    ]
  }
]);

// CRUD operations
await db.add('users', { name: 'John', email: 'john@example.com' });
const users = await db.getAll('users');
await db.update('users', { id: 1, name: 'Jane', email: 'jane@example.com' });
await db.delete('users', 1);
```

### Cache API

The Cache API stores HTTP responses for offline functionality.

```javascript
// Open cache
const cache = await caches.open('my-cache-v1');

// Add resources to cache
await cache.add('/index.html');
await cache.addAll([
  '/index.html',
  '/styles.css',
  '/script.js',
  '/logo.png'
]);

// Put response in cache
const response = await fetch('/api/data');
await cache.put('/api/data', response);

// Match (retrieve) from cache
const cachedResponse = await cache.match('/index.html');
if (cachedResponse) {
  const text = await cachedResponse.text();
  console.log(text);
}

// Delete from cache
await cache.delete('/old-file.js');

// Get all cached requests
const keys = await cache.keys();
console.log(keys); // Array of Request objects

// Delete entire cache
await caches.delete('my-cache-v1');

// Get all cache names
const cacheNames = await caches.keys();
console.log(cacheNames); // ['my-cache-v1', 'my-cache-v2']
```

#### Cache Strategies

```javascript
// Cache-first strategy
async function cacheFirst(request) {
  const cached = await caches.match(request);
  if (cached) return cached;
  
  const response = await fetch(request);
  const cache = await caches.open('my-cache-v1');
  cache.put(request, response.clone());
  
  return response;
}

// Network-first strategy
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    const cache = await caches.open('my-cache-v1');
    cache.put(request, response.clone());
    return response;
  } catch (error) {
    return await caches.match(request);
  }
}

// Stale-while-revalidate
async function staleWhileRevalidate(request) {
  const cached = await caches.match(request);
  
  const fetchPromise = fetch(request).then(response => {
    const cache = caches.open('my-cache-v1');
    cache.put(request, response.clone());
    return response;
  });
  
  return cached || fetchPromise;
}
```

### Storage Quota Management

```javascript
// Check storage quota
if (navigator.storage && navigator.storage.estimate) {
  const estimate = await navigator.storage.estimate();
  console.log('Quota:', estimate.quota);
  console.log('Usage:', estimate.usage);
  console.log('Percentage:', (estimate.usage / estimate.quota * 100).toFixed(2) + '%');
}

// Request persistent storage
if (navigator.storage && navigator.storage.persist) {
  const persistent = await navigator.storage.persist();
  console.log('Persistent storage:', persistent);
}

// Check if storage is persistent
if (navigator.storage && navigator.storage.persisted) {
  const persisted = await navigator.storage.persisted();
  console.log('Is persisted:', persisted);
}
```

---

## 5.4 Fetch API and AJAX

The Fetch API provides a modern interface for making HTTP requests.

### Fetch Basics

```javascript
// Simple GET request
const response = await fetch('https://api.example.com/data');
const data = await response.json();

// With error handling
try {
  const response = await fetch('https://api.example.com/data');
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  const data = await response.json();
  console.log(data);
} catch (error) {
  console.error('Fetch error:', error);
}

// Different response types
const json = await response.json();      // Parse JSON
const text = await response.text();      // Get as text
const blob = await response.blob();      // Get as Blob
const arrayBuffer = await response.arrayBuffer(); // Get as ArrayBuffer
const formData = await response.formData();       // Parse FormData
```

### Request Options

```javascript
// POST request with JSON
const response = await fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  body: JSON.stringify({
    name: 'John',
    email: 'john@example.com'
  })
});

// PUT request
const response = await fetch(`https://api.example.com/users/${id}`, {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(updatedUser)
});

// DELETE request
const response = await fetch(`https://api.example.com/users/${id}`, {
  method: 'DELETE'
});

// With credentials (cookies)
const response = await fetch('https://api.example.com/data', {
  credentials: 'include' // 'same-origin', 'omit', 'include'
});

// Request modes
const response = await fetch(url, {
  mode: 'cors',        // 'cors', 'no-cors', 'same-origin'
  cache: 'no-cache',   // 'default', 'no-cache', 'reload', 'force-cache', 'only-if-cached'
  redirect: 'follow',  // 'follow', 'error', 'manual'
  referrerPolicy: 'no-referrer' // Referrer policy
});
```

### Request and Response Objects

```javascript
// Create Request object
const request = new Request('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ key: 'value' })
});

const response = await fetch(request);

// Response properties
console.log(response.ok);          // true if status 200-299
console.log(response.status);      // 200
console.log(response.statusText);  // "OK"
console.log(response.headers);     // Headers object
console.log(response.url);         // Final URL after redirects
console.log(response.redirected);  // Was redirected?
console.log(response.type);        // 'basic', 'cors', 'opaque'

// Response can only be read once
const data1 = await response.json();
// const data2 = await response.json(); // Error!

// Clone response to read multiple times
const response = await fetch(url);
const clone = response.clone();

const data1 = await response.json();
const data2 = await clone.json();
```

### Headers API

```javascript
// Create headers
const headers = new Headers();
headers.append('Content-Type', 'application/json');
headers.append('Authorization', 'Bearer token');

// Set (replaces existing)
headers.set('Content-Type', 'text/plain');

// Get
console.log(headers.get('Content-Type')); // "text/plain"

// Has
console.log(headers.has('Authorization')); // true

// Delete
headers.delete('Authorization');

// Iterate
for (const [key, value] of headers) {
  console.log(`${key}: ${value}`);
}

// From object
const headers = new Headers({
  'Content-Type': 'application/json',
  'Authorization': 'Bearer token'
});

// Use in fetch
const response = await fetch(url, { headers });
```

### FormData

```javascript
// Create FormData
const formData = new FormData();
formData.append('username', 'john');
formData.append('email', 'john@example.com');

// From form element
const form = document.querySelector('form');
const formData = new FormData(form);

// Append file
const fileInput = document.querySelector('input[type="file"]');
formData.append('avatar', fileInput.files[0]);

// Send FormData
const response = await fetch('https://api.example.com/upload', {
  method: 'POST',
  body: formData // Don't set Content-Type (automatic with boundary)
});

// Iterate FormData
for (const [key, value] of formData) {
  console.log(key, value);
}

// Convert to object
const obj = Object.fromEntries(formData);
```

### URLSearchParams

```javascript
// Create URLSearchParams
const params = new URLSearchParams();
params.append('page', '1');
params.append('limit', '10');
params.append('sort', 'name');

// From object
const params = new URLSearchParams({
  page: 1,
  limit: 10,
  sort: 'name'
});

// From string
const params = new URLSearchParams('page=1&limit=10');

// Use in URL
const url = `https://api.example.com/users?${params}`;
// https://api.example.com/users?page=1&limit=10&sort=name

// Send as body (x-www-form-urlencoded)
const response = await fetch(url, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: params
});

// Iterate
for (const [key, value] of params) {
  console.log(key, value);
}

// Methods
params.get('page');      // "1"
params.has('page');      // true
params.set('page', '2'); // Update
params.delete('page');   // Remove
params.toString();       // "limit=10&sort=name"
```

### AbortController with Fetch

```javascript
// Create AbortController
const controller = new AbortController();
const signal = controller.signal;

// Use in fetch
const response = fetch(url, { signal })
  .then(r => r.json())
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('Fetch aborted');
    } else {
      console.error('Fetch error:', err);
    }
  });

// Abort after 5 seconds
setTimeout(() => controller.abort(), 5000);

// Abort on button click
cancelButton.addEventListener('click', () => {
  controller.abort();
});

// Multiple requests, one controller
const controller = new AbortController();

const requests = [
  fetch('/api/users', { signal: controller.signal }),
  fetch('/api/posts', { signal: controller.signal }),
  fetch('/api/comments', { signal: controller.signal })
];

// Abort all requests
controller.abort();
```

### Fetch Wrapper Utility

```javascript
class FetchClient {
  constructor(baseUrl, defaultOptions = {}) {
    this.baseUrl = baseUrl;
    this.defaultOptions = defaultOptions;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseUrl}${endpoint}`;
    const config = {
      ...this.defaultOptions,
      ...options,
      headers: {
        ...this.defaultOptions.headers,
        ...options.headers
      }
    };
    
    try {
      const response = await fetch(url, config);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const contentType = response.headers.get('content-type');
      if (contentType && contentType.includes('application/json')) {
        return await response.json();
      }
      
      return await response.text();
    } catch (error) {
      console.error('Request failed:', error);
      throw error;
    }
  }
  
  get(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'GET' });
  }
  
  post(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      body: JSON.stringify(data)
    });
  }
  
  put(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      body: JSON.stringify(data)
    });
  }
  
  delete(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'DELETE' });
  }
  
  setHeader(key, value) {
    this.defaultOptions.headers = {
      ...this.defaultOptions.headers,
      [key]: value
    };
  }
}

// Usage
const api = new FetchClient('https://api.example.com', {
  headers: {
    'Authorization': 'Bearer token123'
  }
});

const users = await api.get('/users');
const newUser = await api.post('/users', { name: 'John' });
const updated = await api.put('/users/1', { name: 'Jane' });
await api.delete('/users/1');

// Update token
api.setHeader('Authorization', 'Bearer newtoken456');
```

### CORS (Cross-Origin Resource Sharing)

```javascript
// Simple request (GET, HEAD, POST with simple headers)
const response = await fetch('https://api.example.com/data');

// CORS headers in response
console.log(response.headers.get('Access-Control-Allow-Origin'));
// "*" or "https://yoursite.com"

// Preflight request (PUT, DELETE, custom headers)
// Browser automatically sends OPTIONS request

const response = await fetch('https://api.example.com/users', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'X-Custom-Header': 'value'
  },
  body: JSON.stringify(data)
});

// Server must respond to OPTIONS with:
// Access-Control-Allow-Origin
// Access-Control-Allow-Methods
// Access-Control-Allow-Headers
// Access-Control-Max-Age

// Credentials (cookies) in CORS
const response = await fetch('https://api.example.com/data', {
  credentials: 'include'
});

// Server must respond with:
// Access-Control-Allow-Origin: https://yoursite.com (not *)
// Access-Control-Allow-Credentials: true
```

---

## 5.5 Modern Browser APIs

### Intersection Observer API

Efficiently detect when elements enter/exit the viewport.

```javascript
// Create observer
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Element visible:', entry.target);
      // Load image, start animation, etc.
      entry.target.classList.add('visible');
    } else {
      console.log('Element hidden:', entry.target);
      entry.target.classList.remove('visible');
    }
  });
}, {
  root: null,          // viewport
  rootMargin: '0px',   // margin around root
  threshold: 0.5       // 50% visible
});

// Observe elements
const elements = document.querySelectorAll('.lazy');
elements.forEach(el => observer.observe(el));

// Stop observing
observer.unobserve(element);

// Disconnect all
observer.disconnect();

// Lazy loading images
const imageObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      img.classList.remove('lazy');
      imageObserver.unobserve(img);
    }
  });
});

document.querySelectorAll('img.lazy').forEach(img => {
  imageObserver.observe(img);
});

// Infinite scroll
const sentinelObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadMoreItems();
    }
  });
});

const sentinel = document.querySelector('.sentinel');
sentinelObserver.observe(sentinel);
```

### Mutation Observer API

Watch for DOM changes.

```javascript
// Create observer
const observer = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    console.log('Type:', mutation.type);
    console.log('Target:', mutation.target);
    
    if (mutation.type === 'childList') {
      console.log('Added:', mutation.addedNodes);
      console.log('Removed:', mutation.removedNodes);
    }
    
    if (mutation.type === 'attributes') {
      console.log('Attribute:', mutation.attributeName);
      console.log('Old value:', mutation.oldValue);
    }
  });
});

// Start observing
observer.observe(document.body, {
  childList: true,        // Watch for child additions/removals
  attributes: true,       // Watch for attribute changes
  characterData: true,    // Watch for text content changes
  subtree: true,          // Watch descendants
  attributeOldValue: true,// Record old attribute values
  characterDataOldValue: true, // Record old text values
  attributeFilter: ['class', 'id'] // Only watch specific attributes
});

// Stop observing
observer.disconnect();

// Practical example: Auto-save on content change
const contentObserver = new MutationObserver(() => {
  saveContent();
});

contentObserver.observe(document.querySelector('.editor'), {
  childList: true,
  characterData: true,
  subtree: true
});
```

### Resize Observer API

Detect element size changes.

```javascript
const observer = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    console.log('Element:', entry.target);
    console.log('Content rect:', entry.contentRect);
    console.log('Width:', entry.contentRect.width);
    console.log('Height:', entry.contentRect.height);
    
    // Respond to size change
    if (entry.contentRect.width < 500) {
      entry.target.classList.add('small');
    } else {
      entry.target.classList.remove('small');
    }
  });
});

observer.observe(element);

// Observe multiple elements
document.querySelectorAll('.responsive').forEach(el => {
  observer.observe(el);
});

// Stop observing
observer.unobserve(element);
observer.disconnect();
```

### Performance Observer API

Monitor performance metrics.

```javascript
// Observe performance entries
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach(entry => {
    console.log('Name:', entry.name);
    console.log('Type:', entry.entryType);
    console.log('Start:', entry.startTime);
    console.log('Duration:', entry.duration);
  });
});

// Observe specific entry types
observer.observe({ entryTypes: ['navigation', 'resource', 'measure'] });

// Observe navigation timing
observer.observe({ type: 'navigation', buffered: true });

// Observe resource timing
observer.observe({ type: 'resource', buffered: true });

// User timing (custom marks and measures)
performance.mark('start-task');
// ... do something ...
performance.mark('end-task');
performance.measure('task-duration', 'start-task', 'end-task');

observer.observe({ type: 'measure', buffered: true });

// Disconnect
observer.disconnect();
```

### Web Workers

Run JavaScript in background threads.

```javascript
// Create worker
const worker = new Worker('worker.js');

// Send message to worker
worker.postMessage({ type: 'start', data: [1, 2, 3] });

// Receive message from worker
worker.onmessage = (event) => {
  console.log('Result:', event.data);
};

// Handle errors
worker.onerror = (error) => {
  console.error('Worker error:', error);
};

// Terminate worker
worker.terminate();

// worker.js
self.onmessage = (event) => {
  const { type, data } = event.data;
  
  if (type === 'start') {
    // CPU-intensive task
    const result = processData(data);
    self.postMessage(result);
  }
};

function processData(data) {
  // Heavy computation
  return data.map(x => x * 2);
}
```

### Geolocation API

Access user's location (requires permission).

```javascript
if ('geolocation' in navigator) {
  // Get current position (one-time)
  navigator.geolocation.getCurrentPosition(
    (position) => {
      console.log('Latitude:', position.coords.latitude);
      console.log('Longitude:', position.coords.longitude);
      console.log('Accuracy:', position.coords.accuracy, 'meters');
      console.log('Altitude:', position.coords.altitude);
      console.log('Speed:', position.coords.speed);
    },
    (error) => {
      console.error('Geolocation error:', error.message);
    },
    {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 0
    }
  );
  
  // Watch position (continuous)
  const watchId = navigator.geolocation.watchPosition(
    (position) => {
      console.log('Position updated:', position.coords);
    },
    (error) => {
      console.error('Watch error:', error);
    }
  );
  
  // Stop watching
  navigator.geolocation.clearWatch(watchId);
}
```

### Notification API

Display system notifications (requires permission).

```javascript
// Request permission
async function requestNotificationPermission() {
  const permission = await Notification.requestPermission();
  console.log('Permission:', permission); // 'granted', 'denied', 'default'
  return permission === 'granted';
}

// Show notification
if (Notification.permission === 'granted') {
  new Notification('Hello!', {
    body: 'This is a notification',
    icon: '/icon.png',
    badge: '/badge.png',
    tag: 'unique-id', // Prevents duplicates
    requireInteraction: false,
    silent: false
  });
}

// With event handlers
const notification = new Notification('Title', {
  body: 'Message'
});

notification.onclick = () => {
  console.log('Notification clicked');
  window.focus();
  notification.close();
};

notification.onclose = () => {
  console.log('Notification closed');
};

notification.onerror = () => {
  console.error('Notification error');
};
```

### Clipboard API

Modern clipboard access (requires permission).

```javascript
// Write to clipboard
async function copyToClipboard(text) {
  try {
    await navigator.clipboard.writeText(text);
    console.log('Copied to clipboard');
  } catch (error) {
    console.error('Copy failed:', error);
  }
}

// Read from clipboard
async function pasteFromClipboard() {
  try {
    const text = await navigator.clipboard.readText();
    console.log('Pasted:', text);
    return text;
  } catch (error) {
    console.error('Paste failed:', error);
  }
}

// Copy rich content
async function copyRichContent() {
  const blob = new Blob(['<b>Bold text</b>'], { type: 'text/html' });
  const item = new ClipboardItem({ 'text/html': blob });
  
  await navigator.clipboard.write([item]);
}

// Fallback for older browsers
function fallbackCopy(text) {
  const textarea = document.createElement('textarea');
  textarea.value = text;
  document.body.appendChild(textarea);
  textarea.select();
  document.execCommand('copy');
  document.body.removeChild(textarea);
}
```

### Page Visibility API

Detect when page is visible/hidden.

```javascript
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    console.log('Page hidden');
    // Pause video, stop animations, etc.
    pauseMedia();
  } else {
    console.log('Page visible');
    // Resume activity
    resumeMedia();
  }
});

// Check current state
console.log('Is hidden:', document.hidden);
console.log('Visibility state:', document.visibilityState);
// 'visible', 'hidden', 'prerender'
```

### Broadcast Channel API

Communication between tabs/windows.

```javascript
// Create channel
const channel = new BroadcastChannel('my-channel');

// Send message
channel.postMessage({ type: 'update', data: 'Hello' });

// Receive messages
channel.onmessage = (event) => {
  console.log('Received:', event.data);
};

// Close channel
channel.close();

// Practical example: Sync logout across tabs
const authChannel = new BroadcastChannel('auth');

// Tab 1: User logs out
function logout() {
  localStorage.removeItem('token');
  authChannel.postMessage({ type: 'logout' });
  window.location.href = '/login';
}

// Tab 2: Receives logout message
authChannel.onmessage = (event) => {
  if (event.data.type === 'logout') {
    window.location.href = '/login';
  }
};
```

---

## Comprehensive FAQs

**Q1: Which storage method should I use?**

**A:**
- **Cookies:** Authentication tokens, small data sent to server
- **localStorage:** User preferences, settings (persistent)
- **sessionStorage:** Temporary data, form state (per-tab)
- **IndexedDB:** Large datasets, offline data (complex queries)
- **Cache API:** HTTP responses, offline functionality (PWAs)

---

**Q2: How do I handle CORS errors?**

**A:** CORS is a server-side configuration. The server must send appropriate `Access-Control-Allow-*` headers. Client-side workarounds include proxies or JSONP (deprecated).

```javascript
// ❌ Cannot fix CORS client-side
fetch('https://api.example.com/data', {
  mode: 'no-cors' // ❌ Response will be opaque
});

// ✅ Server must send proper headers
// Access-Control-Allow-Origin: *
// Access-Control-Allow-Methods: GET, POST
```

---

**Q3: When should I use Intersection Observer vs scroll events?**

**A:** Use Intersection Observer for better performance. Scroll events fire frequently and require throttling. Intersection Observer is optimized by the browser.

```javascript
// ❌ Scroll event (needs throttling)
window.addEventListener('scroll', throttle(() => {
  checkVisibility();
}, 100));

// ✅ Intersection Observer (optimized)
const observer = new IntersectionObserver(callback);
observer.observe(element);
```

---

**Q4: How do I prevent memory leaks with observers?**

**A:** Always disconnect or unobserve when elements are removed or components unmount.

```javascript
// Create observer
const observer = new IntersectionObserver(callback);
observer.observe(element);

// Cleanup
element.remove();
observer.unobserve(element); // ✅ Clean up
// or
observer.disconnect(); // ✅ Clean all
```

---

**Q5: What's the difference between fetch() errors and HTTP errors?**

**A:** `fetch()` only rejects on network errors. HTTP errors (404, 500) resolve successfully. Check `response.ok`.

```javascript
try {
  const response = await fetch(url);
  
  if (!response.ok) { // ✅ Check HTTP status
    throw new Error(`HTTP ${response.status}`);
  }
  
  const data = await response.json();
} catch (error) {
  // Network error OR HTTP error
  console.error(error);
}
```

---

## Interview Questions (Storage & APIs)

**Question 1: Implement a localStorage wrapper with expiration.**

**Difficulty:** Mid-Level

```javascript
class Storage {
  static set(key, value, ttl = null) {
    const item = {
      value,
      expiry: ttl ? Date.now() + ttl : null
    };
    localStorage.setItem(key, JSON.stringify(item));
  }
  
  static get(key) {
    const item = localStorage.getItem(key);
    if (!item) return null;
    
    const parsed = JSON.parse(item);
    
    if (parsed.expiry && Date.now() > parsed.expiry) {
      localStorage.removeItem(key);
      return null;
    }
    
    return parsed.value;
  }
}

// Usage
Storage.set('token', 'abc123', 60000); // 1 minute
setTimeout(() => {
  console.log(Storage.get('token')); // null (expired)
}, 61000);
```

---

**Question 2: Explain and implement infinite scroll with Intersection Observer.**

**Difficulty:** Mid-Level

```javascript
class InfiniteScroll {
  constructor(container, loadMore) {
    this.container = container;
    this.loadMore = loadMore;
    this.loading = false;
    
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersect(entries),
      { rootMargin: '100px' }
    );
    
    this.createSentinel();
  }
  
  createSentinel() {
    this.sentinel = document.createElement('div');
    this.sentinel.className = 'sentinel';
    this.container.appendChild(this.sentinel);
    this.observer.observe(this.sentinel);
  }
  
  async handleIntersect(entries) {
    if (entries[0].isIntersecting && !this.loading) {
      this.loading = true;
      
      const hasMore = await this.loadMore();
      
      if (!hasMore) {
        this.observer.disconnect();
      }
      
      this.loading = false;
    }
  }
  
  destroy() {
    this.observer.disconnect();
    this.sentinel.remove();
  }
}

// Usage
const scroll = new InfiniteScroll(
  document.querySelector('.list'),
  async () => {
    const items = await fetchMoreItems();
    renderItems(items);
    return items.length > 0;
  }
);
```

---

**Question 3: Create a fetch wrapper with retry logic and caching.**

**Difficulty:** Senior

```javascript
class FetchWithCache {
  constructor() {
    this.cache = new Map();
  }
  
  async fetch(url, options = {}) {
    const {
      cache = true,
      retry = 3,
      retryDelay = 1000,
      timeout = 5000
    } = options;
    
    // Check cache
    if (cache && this.cache.has(url)) {
      return this.cache.get(url);
    }
    
    // Retry logic
    for (let attempt = 0; attempt <= retry; attempt++) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(
          () => controller.abort(),
          timeout
        );
        
        const response = await fetch(url, {
          ...options,
          signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }
        
        const data = await response.json();
        
        if (cache) {
          this.cache.set(url, data);
        }
        
        return data;
      } catch (error) {
        if (attempt === retry) {
          throw error;
        }
        
        await new Promise(r => setTimeout(r, retryDelay * (attempt + 1)));
      }
    }
  }
  
  clearCache() {
    this.cache.clear();
  }
}
```

---

## Key Takeaways

✅ **Storage:**
- Use appropriate storage for use case
- IndexedDB for large datasets
- Cache API for offline functionality
- Always handle quota limits
- Clean up expired data

✅ **Fetch API:**
- Modern replacement for XMLHttpRequest
- Always check response.ok
- Use AbortController for cancellation
- Handle CORS properly
- Implement retry logic for reliability

✅ **Modern APIs:**
- Intersection Observer for lazy loading
- Mutation Observer for DOM watching
- Resize Observer for responsive layouts
- Web Workers for heavy computation
- Broadcast Channel for tab communication

✅ **Best Practices:**
- Request permissions respectfully
- Provide fallbacks for unsupported APIs
- Clean up observers and listeners
- Handle errors gracefully
- Optimize for performance

---

## Conclusion

Part 5 explored browser APIs and DOM manipulation, covering:
- Efficient DOM manipulation and traversal
- Modern event handling patterns
- Client-side storage options
- Fetch API for network requests
- Modern observer APIs

**What's Next:**

In **Part 6: Node.js and Server-Side JavaScript**, we'll explore:
- Node.js fundamentals and architecture
- Core modules (fs, http, path, etc.)
- NPM and package management
- Express.js and web frameworks
- Database integration
- Authentication and security

Master these browser APIs to build powerful, interactive web applications!

---

**Total Word Count: Part 5 Complete (~20,000 words)**
