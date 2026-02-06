# Functions Best Practices

## Table of Contents

- [Use arrow functions for anonymous functions](#use-arrow-functions-for-anonymous-functions)
- [Keep functions small and focused](#keep-functions-small-and-focused)
- [Use descriptive function names](#use-descriptive-function-names)
- [Limit function parameters](#limit-function-parameters)
- [Use default parameters](#use-default-parameters)
- [Use destructuring for object parameters](#use-destructuring-for-object-parameters)
- [Return early to avoid deep nesting](#return-early-to-avoid-deep-nesting)
- [Avoid side effects](#avoid-side-effects)
- [Use pure functions when possible](#use-pure-functions-when-possible)
- [Document complex functions](#document-complex-functions)

---

## Use arrow functions for anonymous functions

Arrow functions provide cleaner syntax and lexical `this` binding.

```javascript
// ❌ Bad
const numbers = [1, 2, 3];
const doubled = numbers.map(function (num) {
	return num * 2;
});

setTimeout(function () {
	console.log("Done");
}, 1000);

// ✅ Good
const numbers = [1, 2, 3];
const doubled = numbers.map((num) => num * 2);

setTimeout(() => {
	console.log("Done");
}, 1000);

// ✅ Good - Single parameter doesn't need parentheses
const squared = numbers.map((num) => num * num);
```

> **Note:** Use regular functions for object methods when you need access to `this`.

---

## Keep functions small and focused

A function should do one thing and do it well. If it's doing multiple things, break it down.

```javascript
// ❌ Bad - Function does too many things
function processUserData(user) {
	// Validate
	if (!user.email || !user.name) {
		throw new Error("Invalid user");
	}

	// Transform
	const normalizedEmail = user.email.toLowerCase();
	const formattedName = user.name.trim();

	// Save to database
	database.save({ email: normalizedEmail, name: formattedName });

	// Send notification
	emailService.send(normalizedEmail, "Welcome!");

	// Log
	logger.info(`User ${formattedName} created`);
}

// ✅ Good - Separate concerns into smaller functions
function validateUser(user) {
	if (!user.email || !user.name) {
		throw new Error("Invalid user");
	}
}

function normalizeUserData(user) {
	return {
		email: user.email.toLowerCase(),
		name: user.name.trim(),
	};
}

function saveUser(userData) {
	return database.save(userData);
}

function sendWelcomeEmail(email) {
	return emailService.send(email, "Welcome!");
}

function processUserData(user) {
	validateUser(user);

	const normalizedUser = normalizeUserData(user);

	saveUser(normalizedUser);
	sendWelcomeEmail(normalizedUser.email);

	logger.info(`User ${normalizedUser.name} created`);
}
```

---

## Use descriptive function names

Function names should describe what they do. Use verbs for actions.

```javascript
// ❌ Bad
function data() {}
function process() {}
function handle() {}
function doIt() {}
function stuff(user) {}

// ✅ Good
function fetchUserData() {}
function validateEmail() {}
function calculateTotalPrice() {}
function formatDate() {}
function sendNotification(user) {}

// ✅ Good - Boolean functions should ask a question
function isValidEmail(email) {}
function hasPermission(user) {}
function canAccessResource(user, resource) {}
function shouldShowBanner() {}
```

---

## Limit function parameters

Functions with many parameters are hard to use and test. Aim for 3 or fewer.

```javascript
// ❌ Bad - Too many parameters
function createUser(firstName, lastName, email, age, address, phone, role) {
	// ...
}

createUser(
	"John",
	"Doe",
	"john@example.com",
	30,
	"123 Main St",
	"555-1234",
	"admin",
);

// ✅ Good - Use an options object
function createUser({ firstName, lastName, email, age, address, phone, role }) {
	// ...
}

createUser({
	firstName: "John",
	lastName: "Doe",
	email: "john@example.com",
	age: 30,
	address: "123 Main St",
	phone: "555-1234",
	role: "admin",
});
```

---

## Use default parameters

Default parameters make functions more flexible and self-documenting.

```javascript
// ❌ Bad - Manual default handling
function greet(name, greeting) {
	name = name || "Guest";
	greeting = greeting || "Hello";

	return `${greeting}, ${name}!`;
}

// ✅ Good - ES6 default parameters
function greet(name = "Guest", greeting = "Hello") {
	return `${greeting}, ${name}!`;
}

// ✅ Good - Default parameters with objects
function fetchData(url, options = {}) {
	const { method = "GET", headers = {}, timeout = 5000 } = options;
	// ...
}

// ✅ Good - Default parameter expressions
function createId(prefix = "id", timestamp = Date.now()) {
	return `${prefix}_${timestamp}`;
}
```

---

## Use destructuring for object parameters

Destructuring makes it clear what properties a function expects.

```javascript
// ❌ Bad
function displayUser(user) {
	console.log(user.name);
	console.log(user.email);
	console.log(user.role);
}

// ✅ Good - Destructure in parameters
function displayUser({ email, name, role }) {
	console.log(name);
	console.log(email);
	console.log(role);
}

// ✅ Good - With defaults
function displayUser({ email = "", name = "Unknown", role = "user" } = {}) {
	console.log(name);
	console.log(email);
	console.log(role);
}

// ✅ Good - Partial destructuring when needed
function updateUser(userId, { email, name }) {
	// userId is simple, but we only need name and email from the update object
}
```

---

## Return early to avoid deep nesting

Use early returns to reduce indentation and improve readability.

```javascript
// ❌ Bad - Deep nesting
function processPayment(order) {
	if (order) {
		if (order.items.length > 0) {
			if (order.paymentMethod) {
				if (isValidPaymentMethod(order.paymentMethod)) {
					// Process payment
					return processTransaction(order);
				} else {
					throw new Error("Invalid payment method");
				}
			} else {
				throw new Error("No payment method");
			}
		} else {
			throw new Error("Empty order");
		}
	} else {
		throw new Error("No order provided");
	}
}

// ✅ Good - Early returns (Guard clauses)
function processPayment(order) {
	if (!order) {
		throw new Error("No order provided");
	}

	if (order.items.length === 0) {
		throw new Error("Empty order");
	}

	if (!order.paymentMethod) {
		throw new Error("No payment method");
	}

	if (!isValidPaymentMethod(order.paymentMethod)) {
		throw new Error("Invalid payment method");
	}

	return processTransaction(order);
}
```

---

## Avoid side effects

Functions should ideally not modify external state.

```javascript
// ❌ Bad - Modifies external state
let total = 0;

function addToTotal(value) {
	total += value; // Side effect: modifies external variable
}

// ❌ Bad - Modifies input parameter
function addItemToCart(cart, item) {
	cart.push(item); // Mutates the original array

	return cart;
}

// ✅ Good - Returns new value without side effects
function addToTotal(currentTotal, value) {
	return currentTotal + value;
}

// ✅ Good - Returns new array
function addItemToCart(cart, item) {
	return [...cart, item]; // Returns new array
}
```

---

## Use pure functions when possible

Pure functions are predictable, testable, and easier to reason about.

```javascript
// ❌ Bad - Impure function (depends on external state)
let taxRate = 0.1;

function calculateTotal(price) {
	return price + price * taxRate; // Depends on external taxRate
}

// ❌ Bad - Impure function (non-deterministic)
function generateId() {
	return Math.random().toString(36).substr(2, 9);
}

// ✅ Good - Pure function
function calculateTotal(price, taxRate) {
	return price + price * taxRate;
}

// ✅ Good - Pure function for transformation
function formatUserName(firstName, lastName) {
	return `${firstName} ${lastName}`.trim();
}

// ✅ Good - Pure function for filtering
function getActiveUsers(users) {
	return users.filter((user) => user.isActive);
}
```

---

## Document complex functions

Use JSDoc comments for functions with complex logic or non-obvious behavior.

```javascript
// ✅ Good - JSDoc documentation
/**
 * Calculates the discount based on user tier and cart total.
 *
 * @param {Object} user - The user object
 * @param {string} user.tier - User membership tier ('bronze' | 'gold' | 'silver')
 * @param {number} cartTotal - Total cart value in cents
 * @returns {number} Discount amount in cents
 * @throws {Error} If user tier is invalid
 *
 * @example
 * const discount = calculateDiscount({ tier: 'gold' }, 10000);
 * // Returns 1500 (15% of 10000)
 */
function calculateDiscount(user, cartTotal) {
	const discountRates = {
		bronze: 0.05,
		gold: 0.15,
		silver: 0.1
	};

	const rate = discountRates[user.tier];

	if (rate === undefined) {
		throw new Error(`Invalid tier: ${user.tier}`);
	}

	return Math.round(cartTotal * rate);
}
```

---

## Summary

| Rule               | Description                               |
| ------------------ | ----------------------------------------- |
| Arrow functions    | Use for callbacks and anonymous functions |
| Small functions    | Single responsibility, do one thing well  |
| Descriptive names  | `fetchUserData()`, `isValidEmail()`       |
| Limited parameters | 3 or fewer, use objects for more          |
| Default parameters | `function greet(name = 'Guest')`          |
| Destructuring      | `function display({ name, email })`       |
| Early returns      | Guard clauses to reduce nesting           |
| Avoid side effects | Don't modify external state               |
| Pure functions     | Same input → same output                  |
| Documentation      | JSDoc for complex functions               |
