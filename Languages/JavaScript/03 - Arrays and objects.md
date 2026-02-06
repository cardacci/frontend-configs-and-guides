# Arrays and Objects Best Practices

## Table of Contents

- [Use array methods instead of loops](#use-array-methods-instead-of-loops)
- [Use spread operator for copying](#use-spread-operator-for-copying)
- [Use destructuring](#use-destructuring)
- [Use shorthand property names](#use-shorthand-property-names)
- [Use computed property names](#use-computed-property-names)
- [Use optional chaining](#use-optional-chaining)
- [Use nullish coalescing](#use-nullish-coalescing)
- [Avoid mutating arrays and objects](#avoid-mutating-arrays-and-objects)
- [Use Object methods effectively](#use-object-methods-effectively)
- [Use Map and Set when appropriate](#use-map-and-set-when-appropriate)

---

## Use array methods instead of loops

Array methods are more declarative and readable than traditional loops.

```javascript
// ❌ Bad - Imperative loops
const numbers = [1, 2, 3, 4, 5];
const doubled = [];

for (let i = 0; i < numbers.length; i++) {
	doubled.push(numbers[i] * 2);
}

// ✅ Good - map()
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);

// ✅ Good - filter()
const evenNumbers = numbers.filter(num => num % 2 === 0);

// ✅ Good - find()
const users = [{ id: 1, name: 'John' }, { id: 2, name: 'Gabriel' }];
const john = users.find(user => user.name === 'John');

// ✅ Good - some() and every()
const hasAdmin = users.some(user => user.role === 'admin');
const allActive = users.every(user => user.isActive);

// ✅ Good - reduce()
const total = numbers.reduce((sum, num) => sum + num, 0);

// ✅ Good - Chaining methods
const result = users
	.filter(user => user.isActive)
	.map(user => user.name)
	.sort();
```

---

## Use spread operator for copying

The spread operator creates shallow copies without mutating the original.

```javascript
// ❌ Bad - Mutating original array
const original = [1, 2, 3];
const copy = original;
copy.push(4); // Also modifies original!

// ✅ Good - Spread operator for arrays
const original = [1, 2, 3];
const copy = [...original];
const extended = [...original, 4, 5];
const combined = [...array1, ...array2];

// ✅ Good - Spread operator for objects
const user = { name: 'Gabriel', age: 36 };
const copy = { ...user };
const updated = { ...user, age: 37 };
const merged = { ...defaults, ...userSettings };

// ✅ Good - Insert element at specific position
const numbers = [1, 2, 4, 5];
const withThree = [...numbers.slice(0, 2), 3, ...numbers.slice(2)];
// [1, 2, 3, 4, 5]
```

---

## Use destructuring

Destructuring makes code more readable and concise.

```javascript
// ❌ Bad - Accessing properties repeatedly
function displayUser(user) {
	console.log(user.name);
	console.log(user.email);
	console.log(user.address.city);
}

// ✅ Good - Object destructuring
function displayUser(user) {
	const { address: { city }, email, name } = user;

	console.log(name);
	console.log(email);
	console.log(city);
}

// ✅ Good - Array destructuring
const coordinates = [10, 20, 30];
const [x, y, z] = coordinates;

// ✅ Good - Skip elements
const [first, , third] = [1, 2, 3];

// ✅ Good - Rest pattern
const [head, ...tail] = [1, 2, 3, 4, 5];
// head = 1, tail = [2, 3, 4, 5]

// ✅ Good - Default values in destructuring
const { name = 'Anonymous', role = 'user' } = user;

// ✅ Good - Rename while destructuring
const { email: userEmail, name: userName } = user;

// ✅ Good - Swap variables
let a = 1;
let b = 2;

[a, b] = [b, a];
```

---

## Use shorthand property names

When property name matches variable name, use shorthand syntax.

```javascript
// ❌ Bad - Redundant property names
const age = 30;
const email = 'john@example.com';
const name = 'John';

const user = {
	age: age,
	email: email,
	name: name
};

// ✅ Good - Shorthand property names
const user = {
	age,
	email,
	name
};

// ✅ Good - Mix of shorthand and explicit
const user = {
	age,
	createdAt: new Date(),
	email,
	name,
	role: 'admin' // Different name or computed value
};

// ✅ Good - Shorthand methods
const calculator = {
	// ❌ Bad
	add: function(a, b) {
		return a + b;
	},
	
	// ✅ Good
	subtract(a, b) {
		return a - b;
	},
};
```

---

## Use computed property names

ES6 allows dynamic property names in object literals.

```javascript
// ❌ Bad - Two-step process
const key = 'dynamicKey';
const obj = {};

obj[key] = 'value';

// ✅ Good - Computed property names
const key = 'dynamicKey';
const obj = {
	[key]: 'value',
};

// ✅ Good - With expressions
const prefix = 'user';
const obj = {
	[`${prefix}Age`]: 30,
	[`${prefix}Name`]: 'John'
};
// { userAge: 30, userName: 'John' }

// ✅ Good - With functions
const createAction = (type) => ({
	[type]: (payload) => ({ payload, type }),
});
```

---

## Use optional chaining

Optional chaining prevents errors when accessing nested properties that might not exist.

```javascript
// ❌ Bad - Manual null checks
const city = user && user.address && user.address.city;

// ❌ Bad - Ternary chains
const city = user ? (user.address ? user.address.city : undefined) : undefined;

// ✅ Good - Optional chaining
const city = user?.address?.city;

// ✅ Good - With arrays
const firstItem = items?.[0];
const thirdItem = items?.[2]?.name;

// ✅ Good - With function calls
const result = obj?.method?.();

// ✅ Good - Combining with methods
const upperName = user?.name?.toUpperCase();

// ✅ Good - In conditional rendering (React example)
return user?.profile?.avatar && <Avatar src={user.profile.avatar} />;
```

---

## Use nullish coalescing

The `??` operator provides defaults only for `null` and `undefined`, not for falsy values.

```javascript
// ❌ Bad - || treats all falsy values the same
const count = userCount || 10; // 0 becomes 10 (unintended!)
const name = userName || 'Anonymous'; // '' becomes 'Anonymous' (maybe unintended)

// ✅ Good - ?? only for null/undefined
const count = userCount ?? 10; // 0 stays 0
const name = userName ?? 'Anonymous'; // '' stays ''

// ✅ Good - Common use cases
const settings = {
	fontSize: userFontSize ?? 16,
	showSidebar: userShowSidebar ?? true,
	theme: userTheme ?? 'light'
};

// ✅ Good - Combining with optional chaining
const city = user?.address?.city ?? 'Unknown';
const itemCount = response?.data?.items?.length ?? 0;

// ✅ Good - Nullish coalescing assignment
let config = null;

config ??= { default: true }; // Assigns only if null/undefined
```

---

## Avoid mutating arrays and objects

Immutability makes code more predictable and prevents bugs.

```javascript
// ❌ Bad - Mutating array
const items = [1, 2, 3];

items.push(4);
items.splice(1, 1);
items.sort();

// ✅ Good - Create new arrays
const items = [1, 2, 3];
const withNewItem = [...items, 4];
const withoutSecond = items.filter((_, index) => index !== 1);
const sorted = [...items].sort();

// ❌ Bad - Mutating object
const user = { name: 'John', age: 30 };

user.age = 31;

delete user.name;

// ✅ Good - Create new objects
const user = { name: 'John', age: 30 };
const olderUser = { ...user, age: 31 };
const { name, ...userWithoutName } = user;

// ✅ Good - Immutable update patterns
// Update nested property
const state = {
	user: { profile: { name: 'John', age: 30 } }
};

const newState = {
	...state,
	user: {
		...state.user,
		profile: {
			...state.user.profile,
			age: 31,
		},
	},
};

// ✅ Good - Add/update item in array
const addItem = (items, newItem) => [...items, newItem];
const updateItem = (items, id, updates) =>
	items.map(item => item.id === id ? { ...item, ...updates } : item);
const removeItem = (items, id) => items.filter(item => item.id !== id);
```

---

## Use Object methods effectively

JavaScript provides powerful built-in methods for working with objects.

```javascript
// ✅ Good - Object.keys()
const user = { age: 30, email: 'john@example.com', name: 'John' };
const keys = Object.keys(user); // ['age', 'email', 'name']

// ✅ Good - Object.values()
const values = Object.values(user); // [30, 'john@example.com', 'John']

// ✅ Good - Object.entries()
const entries = Object.entries(user);
// [['age', 30], ['email', 'john@example.com'], ['name', 'John']]

// ✅ Good - Iterate over object
Object.entries(user).forEach(([key, value]) => {
	console.log(`${key}: ${value}`);
});

// ✅ Good - Object.fromEntries()
const entries = [['a', 1], ['b', 2]];
const obj = Object.fromEntries(entries); // { a: 1, b: 2 }

// ✅ Good - Transform object
const doubled = Object.fromEntries(
	Object.entries({ a: 1, b: 2 }).map(([key, value]) => [key, value * 2])
);
// { a: 2, b: 4 }

// ✅ Good - Object.assign() for merging
const defaults = { theme: 'light', fontSize: 14 };
const userPrefs = { fontSize: 16 };
const settings = Object.assign({}, defaults, userPrefs);
// { theme: 'light', fontSize: 16 }

// ✅ Good - Check property existence
const hasName = Object.hasOwn(user, 'name'); // true (ES2022+)
const hasName = 'name' in user; // true (includes inherited)
```

---

## Use Map and Set when appropriate

Maps and Sets offer better performance for certain use cases.

```javascript
// ✅ Good - Use Map for dynamic keys
const userCache = new Map();

userCache.set(userId, userData);
userCache.get(userId);
userCache.has(userId);
userCache.delete(userId);

// Map preserves insertion order and allows any key type
const objectKey = { id: 1 };
const map = new Map();

map.set(objectKey, 'value'); // Objects can be keys

// ✅ Good - Use Set for unique values
const uniqueIds = new Set([1, 2, 2, 3, 3, 3]);
// Set { 1, 2, 3 }

uniqueIds.add(4);
uniqueIds.has(2); // true
uniqueIds.delete(1);

// ✅ Good - Remove duplicates from array
const numbers = [1, 2, 2, 3, 3, 3];
const unique = [...new Set(numbers)]; // [1, 2, 3]

// ✅ Good - Set operations
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

// Union
const union = new Set([...setA, ...setB]); // {1, 2, 3, 4}

// Intersection
const intersection = new Set([...setA].filter(x => setB.has(x))); // {2, 3}

// Difference
const difference = new Set([...setA].filter(x => !setB.has(x))); // {1}

// ✅ Good - Convert Map/Set back to array/object
const mapToObject = Object.fromEntries(map);
const setToArray = [...set];
```

---

## Summary

| Pattern | Example |
|---------|---------|
| Array methods | `arr.map()`, `filter()`, `reduce()` |
| Spread copy | `[...arr]`, `{...obj}` |
| Destructuring | `const { name } = user` |
| Shorthand | `{ name, age }` |
| Computed names | `{ [key]: value }` |
| Optional chaining | `user?.address?.city` |
| Nullish coalescing | `value ?? default` |
| Immutable updates | `{ ...obj, prop: newValue }` |
| Object methods | `Object.entries()`, `Object.fromEntries()` |
| Map/Set | Use for dynamic keys and unique values |
