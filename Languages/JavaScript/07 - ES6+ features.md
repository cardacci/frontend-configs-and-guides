# ES6+ Features Best Practices

## Table of Contents

- [Template literals](#template-literals)
- [Destructuring assignment](#destructuring-assignment)
- [Spread and rest operators](#spread-and-rest-operators)
- [Arrow functions](#arrow-functions)
- [Classes](#classes)
- [Modules (import/export)](#modules-importexport)
- [Promises and async/await](#promises-and-asyncawait)
- [Map and Set](#map-and-set)
- [Symbol](#symbol)
- [Optional chaining and nullish coalescing](#optional-chaining-and-nullish-coalescing)
- [New array methods](#new-array-methods)
- [Object enhancements](#object-enhancements)

---

## Template literals

Use template literals for string interpolation and multi-line strings.

```javascript
// ❌ Bad - String concatenation
const greeting = 'Hello, ' + firstName + ' ' + lastName + '!';
const url = baseUrl + '/users/' + userId + '/posts/' + postId;

// ✅ Good - Template literals
const greeting = `Hello, ${firstName} ${lastName}!`;
const url = `${baseUrl}/users/${userId}/posts/${postId}`;

// ✅ Good - Multi-line strings
const html = `
	<div class="card">
		<h2>${title}</h2>
		<p>${description}</p>
	</div>
`;

// ✅ Good - Expressions in templates
const message = `Total: $${(price * quantity).toFixed(2)}`;
const status = `User is ${isActive ? 'active' : 'inactive'}`;

// ✅ Good - Tagged templates for escaping
function html(strings, ...values) {
	return strings.reduce((result, str, i) => {
		const value = values[i] ? escapeHtml(values[i]) : '';

		return result + str + value;
	}, '');
}

const userInput = '<script>alert("xss")</script>';
const safe = html`<div>${userInput}</div>`;
// <div>&lt;script&gt;alert("xss")&lt;/script&gt;</div>
```

---

## Destructuring assignment

Extract values from arrays and objects efficiently.

```javascript
// ❌ Bad - Manual extraction
const user = getUser();
const email = user.email;
const name = user.name;
const role = user.role;

// ✅ Good - Object destructuring
const { name, email, role } = getUser();

// ✅ Good - With defaults
const { name = 'Anonymous', theme = 'light' } = userPreferences;

// ✅ Good - Rename while destructuring
const { id: orderId, name: userName } = response;

// ✅ Good - Nested destructuring
const { 
	user: { email, name },
	meta: { timestamp }
} = apiResponse;

// ✅ Good - Array destructuring
const [first, second, ...rest] = items;
const [x, y] = getCoordinates();

// ✅ Good - Skip elements
const [, , third] = [1, 2, 3]; // third = 3

// ✅ Good - Swap variables
let a = 1;
let b = 2;

[a, b] = [b, a];

// ✅ Good - Function parameters
function createUser({ email, name, role = 'user' }) {
	return { createdAt: new Date(), email, name, role };
}

// ✅ Good - Import destructuring
const { useCallback, useEffect, useState } = React;
```

---

## Spread and rest operators

Use spread for expanding and rest for collecting.

```javascript
// ✅ Good - Spread arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// ✅ Good - Clone arrays
const original = [1, 2, 3];
const copy = [...original];

// ✅ Good - Spread objects
const defaults = { lang: 'en', theme: 'light' };
const userPrefs = { theme: 'dark' };
const settings = { ...defaults, ...userPrefs }; // lang: 'en', theme: 'dark'

// ✅ Good - Add/update properties immutably
const user = { age: 36, name: 'Gabriel' };
const updated = { ...user, age: 37, city: 'Mar del Plata' };

// ✅ Good - Rest parameters
function sum(...numbers) {
	return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4); // 10

// ✅ Good - Rest in destructuring
const { id, ...otherProps } = user;
const [head, ...tail] = [1, 2, 3, 4];

// ✅ Good - Function argument spreading
const args = [1, 2, 3];
Math.max(...args); // 3

// ✅ Good - Converting iterables to arrays
const chars = [...'hello']; // ['h', 'e', 'l', 'l', 'o']
const unique = [...new Set([1, 1, 2, 2, 3])]; // [1, 2, 3]
```

---

## Arrow functions

Use arrow functions for concise syntax and lexical `this`.

```javascript
// ✅ Good - Concise syntax for callbacks
const doubled = numbers.map(n => n * 2);
const active = users.filter(u => u.isActive);
const names = users.map(u => u.name);

// ✅ Good - Implicit return for single expressions
const add = (a, b) => a + b;
const isEven = n => n % 2 === 0;

// ✅ Good - Explicit return for complex logic
const processUser = user => {
	const normalizedName = user.name.trim().toLowerCase();
	const email = user.email.toLowerCase();

	return { ...user, email, name: normalizedName };
};

// ✅ Good - Return object literal (wrap in parentheses)
const createUser = (name, email) => ({ createdAt: Date.now(), email, name });

// ✅ Good - Lexical this binding
class Counter {
	constructor() {
		this.count = 0;
	}
	
	start() {
		// Arrow function preserves 'this'
		setInterval(() => {
			this.count++;
			console.log(this.count);
		}, 1000);
	}
}

// ⚠️ When NOT to use arrow functions
// Object methods (when you need 'this')
const obj = {
	value: 42,
	// ❌ Bad - arrow function doesn't have its own 'this'
	getValue: () => this.value, // undefined
	
	// ✅ Good - regular function
	getValue() {
		return this.value;
	},
};

// Prototype methods
function Person(name) {
	this.name = name;
}

// ✅ Good - regular function for prototype
Person.prototype.greet = function() {
	return `Hello, ${this.name}`;
};
```

---

## Classes

Use ES6 classes for object-oriented patterns.

```javascript
// ✅ Good - Class declaration
class User {
	// Private fields (ES2022+)
	#password;
	
	// Public fields
	role = 'user';
	
	// Static properties
	static DEFAULT_ROLE = 'user';
	static userCount = 0;
	
	constructor(name, email, password) {
		this.#password = password;
		this.email = email;
		this.name = name;
		User.userCount++;
	}
	
	// Instance methods
	greet() {
		return `Hello, ${this.name}!`;
	}
	
	// Getters
	get displayName() {
		return `${this.name} (${this.role})`;
	}
	
	// Setters
	set displayName(value) {
		this.name = value.split(' (')[0];
	}
	
	// Private methods
	#hashPassword(password) {
		return `hashed_${password}`;
	}
	
	// Static methods
	static createGuest() {
		return new User('Guest', 'guest@example.com', '');
	}
}

// ✅ Good - Inheritance
class AdminUser extends User {
	constructor(name, email, password) {
		super(name, email, password);

		this.role = 'admin';
	}
	
	// Override method
	greet() {
		return `${super.greet()} You have admin privileges.`;
	}
	
	deleteUser(userId) {
		// Admin-specific method
	}
}

// Usage
const admin = new AdminUser('John', 'john@example.com', 'secret');

console.log(admin.greet());
console.log(User.userCount);
```

---

## Modules (import/export)

Use ES modules for code organization.

```javascript
// ✅ Good - Named exports (utils.js)
export const PI = 3.14159;

export function add(a, b) {
	return a + b;
}

export function subtract(a, b) {
	return a - b;
}

export class Calculator {
	// ...
}

// ✅ Good - Named imports
import { add, PI, subtract } from './utils.js';
import { add as sum } from './utils.js'; // Rename

// ✅ Good - Default export (User.js)
export default class User {
	constructor(name) {
		this.name = name;
	}
}

// ✅ Good - Default import
import User from './User.js';
import MyUser from './User.js'; // Can use any name

// ✅ Good - Mixed imports
import React, { useEffect, useState } from 'react';

// ✅ Good - Import all as namespace
import * as utils from './utils.js';

utils.add(1, 2);
utils.PI;

// ✅ Good - Re-export
export { User } from './User.js';
export * from './utils.js';
export { default as User } from './User.js';

// ✅ Good - Dynamic imports (code splitting)
async function loadModule() {
	const { heavyFunction } = await import('./heavy-module.js');

	return heavyFunction();
}

// ✅ Good - Conditional imports
let crypto;

if (typeof window === 'undefined') {
	crypto = await import('crypto');
} else {
	crypto = window.crypto;
}
```

---

## Promises and async/await

Modern async patterns for cleaner code.

```javascript
// ✅ Good - Creating Promises
function delay(ms) {
	return new Promise(resolve => setTimeout(resolve, ms));
}

function fetchUser(id) {
	return new Promise((resolve, reject) => {
		api.getUser(id, (error, user) => {
			if (error) {
				reject(error);
			} else {
				resolve(user);
			}
		});
	});
}

// ✅ Good - Promise.all for parallel execution
const [user, posts, comments] = await Promise.all([
	fetchUser(userId),
	fetchPosts(userId),
	fetchComments(userId),
]);

// ✅ Good - Promise.allSettled for partial failure handling
const results = await Promise.allSettled([
	fetchCriticalData(),
	fetchOptionalData(),
]);

const data = results.map(r => 
	r.status === 'fulfilled' ? r.value : null
);

// ✅ Good - Promise.race for timeout
async function fetchWithTimeout(url, timeout = 5000) {
	const controller = new AbortController();
	
	const timeoutPromise = new Promise((_, reject) => {
		setTimeout(() => {
			controller.abort();
			reject(new Error('Request timeout'));
		}, timeout);
	});
	
	return Promise.race([
		fetch(url, { signal: controller.signal }),
		timeoutPromise,
	]);
}

// ✅ Good - Async/await with proper error handling
async function processData() {
	try {
		const rawData = await fetchData();
		const processed = await transform(rawData);

		await save(processed);

		return processed;
	} catch (error) {
		console.error('Processing failed:', error);

		throw error;
	}
}

// ✅ Good - Promise.any (first fulfilled wins)
const fastest = await Promise.any([
	fetch('https://api1.example.com/data'),
	fetch('https://api2.example.com/data'),
	fetch('https://api3.example.com/data')
]);
```

---

## Map and Set

Use Map and Set for specific use cases.

```javascript
// ✅ Good - Map for key-value pairs with any key type
const userCache = new Map();

// Any type as key
userCache.set(1, { name: 'Gabriel' });
userCache.set('admin', { name: 'Admin' });
userCache.set(userObject, { permissions: ['read', 'write'] });

// Map methods
userCache.get(1);           // { name: 'Gabriel' }
userCache.has('admin');     // true
userCache.delete(1);
userCache.size;             // 2
userCache.clear();

// ✅ Good - Iterate over Map
for (const [key, value] of userCache) {
	console.log(key, value);
}

// Map to array
const entries = [...userCache.entries()];
const keys = [...userCache.keys()];
const values = [...userCache.values()];

// ✅ Good - WeakMap for object keys (allows garbage collection)
const privateData = new WeakMap();

class User {
	constructor(name, password) {
		this.name = name;

		privateData.set(this, { password });
	}
	
	checkPassword(attempt) {
		return privateData.get(this).password === attempt;
	}
}

// ✅ Good - Set for unique values
const uniqueIds = new Set([1, 2, 2, 3, 3, 3]);
// Set { 1, 2, 3 }

uniqueIds.add(4);
uniqueIds.has(2);    // true
uniqueIds.delete(1);
uniqueIds.size;      // 3

// ✅ Good - Remove duplicates
const numbers = [1, 1, 2, 2, 3, 3];
const unique = [...new Set(numbers)]; // [1, 2, 3]

// ✅ Good - Set operations
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

// Union
const union = new Set([...setA, ...setB]);

// Intersection
const intersection = new Set([...setA].filter(x => setB.has(x)));

// Difference
const difference = new Set([...setA].filter(x => !setB.has(x)));
```

---

## Symbol

Use Symbols for unique identifiers and well-known behaviors.

```javascript
// ✅ Good - Create unique property keys
const id = Symbol('id');
const name = Symbol('name');

const user = {
	[id]: 123,
	[name]: 'John',
	publicName: 'John Doe',
};

// Symbols don't appear in normal iteration
Object.keys(user);        // ['publicName']
Object.getOwnPropertySymbols(user); // [Symbol(id), Symbol(name)]

// ✅ Good - Symbol.for for global symbols
const globalId = Symbol.for('app.userId');
const sameId = Symbol.for('app.userId');

globalId === sameId; // true

// ✅ Good - Well-known symbols
class Collection {
	constructor(items) {
		this.items = items;
	}
	
	// Make iterable
	[Symbol.iterator]() {
		const items = this.items;
		let index = 0;
		
		return {
			next() {
				if (index < items.length) {
					return {
						value: items[index++], done: false
					};
				}

				return { done: true };
			}
		};
	}
	
	// Custom string tag
	get [Symbol.toStringTag]() {
		return 'Collection';
	}
}

const collection = new Collection([1, 2, 3]);

for (const item of collection) {
	console.log(item);
}
console.log(collection.toString()); // [object Collection]

// ✅ Good - Symbol for private-like properties
const _private = Symbol('private');

class MyClass {
	constructor() {
		this[_private] = 'secret';
	}
	
	getPrivate() {
		return this[_private];
	}
}
```

---

## Optional chaining and nullish coalescing

Safe property access and default values.

```javascript
// ✅ Good - Optional chaining (?.)
const city = user?.address?.city;
const firstItem = items?.[0];
const result = obj?.method?.();

// ✅ Good - Combining with method calls
const upperName = user?.name?.toUpperCase?.();

// ✅ Good - Nullish coalescing (??)
// Only for null/undefined, not falsy values
const count = data.count ?? 0;       // 0 stays 0
const name = data.name ?? 'Anonymous'; // '' stays ''
const enabled = data.enabled ?? true;  // false stays false

// ✅ Good - Combining both
const userName = user?.profile?.name ?? 'Guest';
const itemCount = response?.data?.items?.length ?? 0;

// ✅ Good - Nullish assignment (??=)
let config = null;

config ??= { defaults: true }; // config = { defaults: true }

// ✅ Good - Logical AND assignment (&&=)
let value = 'hello';

value &&= value.toUpperCase(); // 'HELLO'

// ✅ Good - Logical OR assignment (||=)
let name = '';

name ||= 'Default'; // 'Default' (because '' is falsy)

// ⚠️ Difference between || and ??
const value1 = 0 || 10;   // 10 (0 is falsy)
const value2 = 0 ?? 10;   // 0 (0 is not null/undefined)

const value3 = '' || 'default';   // 'default'
const value4 = '' ?? 'default';   // ''
```

---

## New array methods

Modern array methods for cleaner code.

```javascript
// ✅ Good - Array.from()
const nodeList = document.querySelectorAll('div');
const divArray = Array.from(nodeList);

// With mapping function
const numbers = Array.from({ length: 5 }, (_, i) => i + 1);
// [1, 2, 3, 4, 5]

// ✅ Good - Array.of()
const arr = Array.of(1, 2, 3); // [1, 2, 3]

// ✅ Good - find() and findIndex()
const users = [{ id: 1, name: 'John' }, { id: 2, name: 'Jane' }];
const john = users.find(u => u.name === 'John');
const johnIndex = users.findIndex(u => u.name === 'John');

// ✅ Good - findLast() and findLastIndex() (ES2023)
const lastEven = [1, 2, 3, 4, 5].findLast(n => n % 2 === 0); // 4

// ✅ Good - includes()
const hasApple = fruits.includes('apple');

// ✅ Good - flat() and flatMap()
const nested = [[1, 2], [3, 4], [5]];
const flat = nested.flat(); // [1, 2, 3, 4, 5]

const sentences = ['Hello world', 'Foo bar'];
const words = sentences.flatMap(s => s.split(' '));
// ['Hello', 'world', 'Foo', 'bar']

// ✅ Good - at() for negative indexing
const arr = [1, 2, 3, 4, 5];

arr.at(-1);  // 5
arr.at(-2);  // 4

// ✅ Good - toSorted(), toReversed(), toSpliced() (ES2023)
// Non-mutating versions
const original = [3, 1, 2];
const sorted = original.toSorted();     // [1, 2, 3]
const reversed = original.toReversed(); // [2, 1, 3]
// original unchanged: [3, 1, 2]

// ✅ Good - with() for immutable update (ES2023)
const arr = ['a', 'b', 'c'];
const updated = arr.with(1, 'B'); // ['a', 'B', 'c']
```

---

## Object enhancements

Modern object syntax and methods.

```javascript
// ✅ Good - Shorthand properties
const name = 'John';
const age = 30;
const user = { age, name }; // { age: 30, name: 'John' }

// ✅ Good - Shorthand methods
const obj = {
	greet() {
		return 'Hello!';
	},

	async fetchData() {
		return await api.get('/data');
	},

	*generateIds() {
		let id = 0;

		while (true) {
			yield id++;
		}
	},
};

// ✅ Good - Computed property names
const prefix = 'user';
const obj = {
	[`${prefix}Age`]: 30,
	[`${prefix}Name`]: 'John',
	[`get${prefix.charAt(0).toUpperCase() + prefix.slice(1)}`]() {
		return this.userName;
	},
};

// ✅ Good - Object.entries(), Object.values(), Object.keys()
const user = { age: 30, name: 'John' };

Object.keys(user);    // ['age', 'name']
Object.values(user);  // [30, 'John']
Object.entries(user); // [['age', 30], ['name', 'John']]

// ✅ Good - Object.fromEntries()
const entries = [['a', 1], ['b', 2]];
const obj = Object.fromEntries(entries); // { a: 1, b: 2 }

// Transform object
const doubled = Object.fromEntries(
	Object.entries({ a: 1, b: 2 }).map(([k, v]) => [k, v * 2])
);
// { a: 2, b: 4 }

// ✅ Good - Object.hasOwn() (ES2022)
Object.hasOwn(user, 'name'); // true (preferred over hasOwnProperty)

// ✅ Good - Object.groupBy() (ES2024)
const people = [
	{ name: 'John', age: 25 },
	{ name: 'Jane', age: 30 },
	{ name: 'Bob', age: 25 }
];

const byAge = Object.groupBy(people, person => person.age);
// { 25: [{name: 'John'...}, {name: 'Bob'...}], 30: [{name: 'Jane'...}] }
```

---

## Summary

| Feature | Use Case |
|---------|----------|
| Template literals | String interpolation, multi-line |
| Destructuring | Extract values from objects/arrays |
| Spread/Rest | Clone, merge, collect arguments |
| Arrow functions | Callbacks, preserve `this` |
| Classes | Object-oriented patterns |
| Modules | Code organization |
| async/await | Asynchronous code |
| Map/Set | Key-value pairs, unique values |
| Symbol | Unique identifiers |
| `?.` and `??` | Safe access and defaults |
| New array methods | `find`, `flat`, `at`, `toSorted` |
| Object enhancements | Shorthand, computed props |
