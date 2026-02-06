# Variables and Constants Best Practices

## Table of Contents

- [Use `const` by default](#use-const-by-default)
- [Use `let` only when reassignment is needed](#use-let-only-when-reassignment-is-needed)
- [Never use `var`](#never-use-var)
- [Meaningful and descriptive names](#meaningful-and-descriptive-names)
- [Use camelCase for variables and functions](#use-camelcase-for-variables-and-functions)
- [Use UPPER_SNAKE_CASE for true constants](#use-upper_snake_case-for-true-constants)
- [Declare variables at the top of their scope](#declare-variables-at-the-top-of-their-scope)
- [One variable per declaration](#one-variable-per-declaration)
- [Initialize variables when declaring](#initialize-variables-when-declaring)

---

## Use `const` by default

Always use `const` unless you know the variable will be reassigned. This makes your code more predictable and easier to reason about.

```javascript
// ❌ Bad
let apiUrl = "https://api.example.com";
let maxRetries = 3;

// ✅ Good
const apiUrl = "https://api.example.com";
const maxRetries = 3;
```

---

## Use `let` only when reassignment is needed

Only use `let` when you explicitly need to reassign the variable.

```javascript
// ✅ Good - let is appropriate here
let count = 0;

for (let i = 0; i < items.length; i++) {
	count += items[i].value;
}

// ✅ Good - let for variables that change
let isLoading = true;

fetchData().then(() => {
	isLoading = false;
});
```

---

## Never use `var`

`var` has function scope and hoisting issues that can lead to bugs. Always use `const` or `let`.

```javascript
// ❌ Bad - var has unexpected hoisting behavior
function example() {
	console.log(x); // undefined (hoisted)
	var x = 5;

	if (true) {
		var y = 10; // y is function-scoped, not block-scoped
	}

	console.log(y); // 10 (unexpected)
}

// ✅ Good - let/const have block scope
function example() {
	// console.log(x); // ReferenceError (not hoisted)
	const x = 5;

	if (true) {
		const y = 10; // y is block-scoped
	}

	// console.log(y); // ReferenceError (as expected)
}
```

---

## Meaningful and descriptive names

Variable names should clearly describe what they contain.

```javascript
// ❌ Bad
const d = new Date();
const n = "John";
const arr = [1, 2, 3];
const obj = { name: "Product", price: 100 };
const flag = true;

// ✅ Good
const currentDate = new Date();
const userName = "John";
const productIds = [1, 2, 3];
const productDetails = { name: "Product", price: 100 };
const isAuthenticated = true;
```

---

## Use camelCase for variables and functions

Follow JavaScript naming conventions consistently.

```javascript
// ❌ Bad
const user_name = "John";
const UserAge = 25;
const ISACTIVE = true;
function GetUserData() {}

// ✅ Good
const userName = "John";
const userAge = 25;
const isActive = true;
function getUserData() {}
```

---

## Use UPPER_SNAKE_CASE for true constants

Use uppercase with underscores for values that are truly constant and known at compile time.

```javascript
// ✅ Good - True constants (known at compile time)
const API_BASE_URL = "https://api.example.com";
const HTTP_STATUS_OK = 200;
const MAX_ITEMS_PER_PAGE = 50;
const MILLISECONDS_PER_SECOND = 1000;

// ✅ Good - Runtime constants use camelCase
const currentUser = await fetchUser();
const startTime = Date.now();
```

---

## Declare variables at the top of their scope

Declare variables at the beginning of their scope for better readability.

```javascript
// ❌ Bad
function processOrder(order) {
	validateOrder(order);
	const tax = calculateTax(order);
	sendNotification(order);
	const total = order.subtotal + tax;
	const discount = getDiscount(order);
	return total - discount;
}

// ✅ Good
function processOrder(order) {
	const discount = getDiscount(order);
	const tax = calculateTax(order);
	const total = order.subtotal + tax;

	validateOrder(order);
	sendNotification(order);

	return total - discount;
}
```

---

## One variable per declaration

Declare each variable on its own line for better readability and easier version control diffs.

```javascript
// ❌ Bad
const firstName = "John",
	lastName = "Doe",
	age = 30;

let a = 1,
	b = 2,
	c = 3;

// ✅ Good
const age = 30;
const firstName = "John";
const lastName = "Doe";

let a = 1;
let b = 2;
let c = 3;
```

---

## Initialize variables when declaring

Always initialize variables when you declare them to avoid undefined values.

```javascript
// ❌ Bad
let config;
let items;
let userName;

// ... later in code
config = {};
items = [];
userName = "John";

// ✅ Good
let config = {};
let items = [];
let userName = "";

// Or better with const if not reassigned
const config = {};
const items = [];
const userName = "John";
```

---

## Summary

| Rule                           | Example                            |
| ------------------------------ | ---------------------------------- |
| Default to `const`             | `const apiUrl = '...'`             |
| Use `let` for reassignment     | `let count = 0`                    |
| Never use `var`                | ~~`var x = 1`~~                    |
| Descriptive names              | `isAuthenticated`, `userList`      |
| camelCase for variables        | `userName`, `totalPrice`           |
| UPPER_SNAKE_CASE for constants | `MAX_RETRIES`, `API_URL`           |
| One variable per line          | Each `const`/`let` on its own line |
| Initialize when declaring      | `let items = []`                   |
