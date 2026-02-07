# Clean Code Principles for JavaScript

## Table of Contents

- [Write self-documenting code](#write-self-documenting-code)
- [Keep it simple (KISS)](#keep-it-simple-kiss)
- [Don't repeat yourself (DRY)](#dont-repeat-yourself-dry)
- [Single responsibility principle](#single-responsibility-principle)
- [Avoid magic numbers and strings](#avoid-magic-numbers-and-strings)
- [Use meaningful boolean conditions](#use-meaningful-boolean-conditions)
- [Limit function and file length](#limit-function-and-file-length)
- [Consistent code formatting](#consistent-code-formatting)
- [Avoid deep nesting](#avoid-deep-nesting)
- [Write comments that explain why](#write-comments-that-explain-why)

---

## Write self-documenting code

Code should explain itself through clear naming and structure.

```javascript
// ❌ Bad - Unclear what's happening
const x = u.filter(i => i.a && i.d < Date.now() - 86400000);

// ✅ Good - Self-documenting
const ONE_DAY_IN_MS = 24 * 60 * 60 * 1000;
const oneDayAgo = Date.now() - ONE_DAY_IN_MS;

const recentActiveUsers = users.filter(user => 
	user.isActive && user.lastLoginDate < oneDayAgo
);

// ❌ Bad - Cryptic function
function proc(d) {
	return d.map(i => ({ ...i, t: i.p * i.q }));
}

// ✅ Good - Clear purpose
function calculateOrderTotals(orderItems) {
	return orderItems.map(item => ({
		...item,
		total: item.price * item.quantity,
	}));
}

// ❌ Bad - Boolean meaning unclear
if (user.s === 1) {
	// ...
}

// ✅ Good - Boolean meaning is clear
const USER_STATUS = {
	ACTIVE: 1,
	INACTIVE: 0,
	SUSPENDED: 2,
};

if (user.status === USER_STATUS.ACTIVE) {
	// ...
}
```

---

## Keep it simple (KISS)

Choose the simplest solution that works. Avoid over-engineering.

```javascript
// ❌ Bad - Over-engineered
class StringValidator {
	constructor(rules) {
		this.rules = rules;
	}
	
	addRule(rule) {
		this.rules.push(rule);
		return this;
	}
	
	validate(value) {
		return this.rules.every(rule => rule.validate(value));
	}
}

const emailValidator = new StringValidator([])
	.addRule(new RequiredRule())
	.addRule(new EmailFormatRule());

const isValid = emailValidator.validate(email);

// ✅ Good - Simple and effective
function isValidEmail(email) {
	if (!email) {
		return false;
	}

	return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

const isValid = isValidEmail(email);

// ❌ Bad - Unnecessary abstraction
const add = (a) => (b) => a + b;
const increment = add(1);
const result = increment(5);

// ✅ Good - Direct and clear
const result = 5 + 1;
// Or if reuse is needed:
function increment(value) {
	return value + 1;
}

// ❌ Bad - Complex ternary
const status = isAdmin ? 'admin' : isModerator ? 'moderator' : isUser ? 'user' : 'guest';

// ✅ Good - Clear conditionals
function getUserRole(user) {
	if (user.isAdmin) {
		return 'admin';
	}

	if (user.isModerator) {
		return 'moderator';
	}

	if (user.isAuthenticated) {
		return 'user';
	}

	return 'guest';
}
```

---

## Don't repeat yourself (DRY)

Extract common logic into reusable functions or modules.

```javascript
// ❌ Bad - Repeated logic
function validateCreateUser(data) {
	const errors = [];
	
	if (!data.email) {
		errors.push('Email is required');
	} else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email)) {
		errors.push('Invalid email format');
	}
	
	if (!data.name) {
		errors.push('Name is required');
	} else if (data.name.length < 2) {
		errors.push('Name must be at least 2 characters');
	}
	
	return errors;
}

function validateUpdateUser(data) {
	const errors = [];
	
	if (data.email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email)) {
		errors.push('Invalid email format');
	}
	
	if (data.name && data.name.length < 2) {
		errors.push('Name must be at least 2 characters');
	}
	
	return errors;
}

// ✅ Good - Reusable validation functions
const validators = {
	email: (value) => {
		if (!value) {
			return null;
		}

		return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) 
			? null 
			: 'Invalid email format';
	},
	
	required: (fieldName) => (value) => {
		return value ? null : `${fieldName} is required`;
	},
	
	minLength: (min, fieldName) => (value) => {
		if (!value) {
			return null;
		}

		return value.length >= min 
			? null 
			: `${fieldName} must be at least ${min} characters`;
	}
};

function validate(data, rules) {
	const errors = [];

	for (const [field, fieldRules] of Object.entries(rules)) {
		for (const rule of fieldRules) {
			const error = rule(data[field]);

			if (error) {
				errors.push(error);
			}
		}
	}
	
	return errors;
}

// Usage
const createUserRules = {
	email: [validators.required('Email'), validators.email],
	name: [validators.required('Name'), validators.minLength(2, 'Name')],
};

const updateUserRules = {
	email: [validators.email],
	name: [validators.minLength(2, 'Name')],
};
```

---

## Single responsibility principle

Each function or module should do one thing well.

```javascript
// ❌ Bad - Function does too many things
async function handleUserRegistration(formData) {
	// Validate
	if (!formData.email || !formData.password) {
		return { error: 'Missing fields' };
	}
	
	// Hash password
	const hashedPassword = await bcrypt.hash(formData.password, 10);
	
	// Save to database
	const user = await db.users.create({
		email: formData.email,
		password: hashedPassword,
	});
	
	// Send welcome email
	await emailService.send({
		body: 'Thanks for registering...',
		subject: 'Welcome!',
		to: user.email
	});
	
	// Log analytics
	analytics.track('user_registered', { userId: user.id });
	
	return { success: true, user };
}

// ✅ Good - Separate responsibilities
function validateRegistrationData(formData) {
	const errors = [];

	if (!formData.email) {
		errors.push('Email is required');
	}

	if (!formData.password) {
		errors.push('Password is required');
	}

	return errors;
}

async function hashPassword(password) {
	return bcrypt.hash(password, 10);
}

async function createUser(email, hashedPassword) {
	return db.users.create({ email, password: hashedPassword });
}

async function sendWelcomeEmail(user) {
	return emailService.send({
		body: 'Thanks for registering...',
		subject: 'Welcome!',
		to: user.email
	});
}

function trackRegistration(user) {
	analytics.track('user_registered', { userId: user.id });
}

// Orchestrating function
async function handleUserRegistration(formData) {
	const errors = validateRegistrationData(formData);
	if (errors.length > 0) {
		return { error: errors };
	}
	
	const hashedPassword = await hashPassword(formData.password);
	const user = await createUser(formData.email, hashedPassword);
	
	// Non-blocking side effects
	Promise.all([
		sendWelcomeEmail(user),
		trackRegistration(user),
	]).catch(console.error);
	
	return { success: true, user };
}
```

---

## Avoid magic numbers and strings

Use named constants for values that have meaning.

```javascript
// ❌ Bad - Magic numbers and strings
if (user.role === 1) {
	// ...
}

if (response.status === 200) {
	// ...
}

setTimeout(callback, 86400000);

if (password.length < 8) {
	// ...
}

const discount = total * 0.15;

// ✅ Good - Named constants
const USER_ROLES = {
	ADMIN: 2,
	GUEST: 0,
	USER: 1,
};

const HTTP_STATUS = {
	BAD_REQUEST: 400,
	CREATED: 201,
	NOT_FOUND: 404,
	OK: 200,
	UNAUTHORIZED: 401,
};

const ONE_DAY_MS = 24 * 60 * 60 * 1000;

const PASSWORD_MIN_LENGTH = 8;

const DISCOUNT_RATES = {
	PREMIUM: 0.10,
	STANDARD: 0.05,
	VIP: 0.15,
};

// Usage
if (user.role === USER_ROLES.USER) {
	// ...
}

if (response.status === HTTP_STATUS.OK) {
	// ...
}

setTimeout(callback, ONE_DAY_MS);

if (password.length < PASSWORD_MIN_LENGTH) {
	// ...
}

const discount = total * DISCOUNT_RATES.VIP;
```

---

## Use meaningful boolean conditions

Make boolean logic readable and intention-clear.

```javascript
// ❌ Bad - Complex inline conditions
if (user && user.age >= 18 && user.country === 'US' && !user.banned && user.verified) {
	// ...
}

// ✅ Good - Extract to meaningful variables
const isAdult = user.age >= 18;
const isInGoodStanding = !user.banned && user.verified;
const isUSResident = user.country === 'US';
const canAccessContent = user && isAdult && isUSResident && isInGoodStanding;

if (canAccessContent) {
	// ...
}

// ❌ Bad - Negative conditions are confusing
if (!isNotLoggedIn) {
	// Double negative - confusing!
}

// ✅ Good - Positive conditions
if (isLoggedIn) {
	// Clear!
}

// ❌ Bad - Complex ternary
const access = user ? (user.admin ? 'full' : (user.member ? 'limited' : 'none')) : 'none';

// ✅ Good - Function with clear logic
function getAccessLevel(user) {
	if (!user) {
		return 'none';
	}

	if (user.admin) {
		return 'full';
	}

	if (user.member) {
		return 'limited';
	}

	return 'none';
}

// ✅ Good - Boolean function names ask questions
function canUserEdit(user, document) {
	const isAdmin = user.role === 'admin';
	const isNotLocked = !document.isLocked;
	const isOwner = document.authorId === user.id;
	
	return (isAdmin || isOwner) && isNotLocked;
}
```

---

## Limit function and file length

Keep functions short and files focused.

```javascript
// ❌ Bad - Function too long (abbreviated)
async function processCheckout(cart, user, paymentInfo) {
	// 200+ lines of code doing validation, payment processing,
	// inventory updates, email sending, logging...
}

// ✅ Good - Break into smaller functions
async function processCheckout(cart, user, paymentInfo) {
	validateCart(cart);
	validatePaymentInfo(paymentInfo);
	
	const order = await createOrder(cart, user);

	await processPayment(order, paymentInfo);
	await updateInventory(order);
	await sendConfirmationEmail(user, order);
	
	return order;
}

// Each function is small and focused:
function validateCart(cart) {
	if (!cart.items?.length) {
		throw new ValidationError('Cart is empty');
	}
	
	for (const item of cart.items) {
		validateCartItem(item);
	}
}

function validateCartItem(item) {
	if (!item.productId) {
		throw new ValidationError('Invalid cart item');
	}

	if (item.quantity <= 0) {
		throw new ValidationError('Invalid quantity');
	}
}

// ✅ Good - Keep files focused
// Instead of one large utils.js, split into:
// - utils/date.js
// - utils/formatting.js
// - utils/string.js
// - utils/validation.js
```

### File Organization Guidelines

```
// ✅ Good file structure
src/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.styles.js
│   │   └── Button.test.js
│   └── Form/
│       ├── Form.jsx
│       └── Form.test.js
├── hooks/
│   ├── useAuth.js
│   └── useFetch.js
├── services/
│   ├── api.js
│   └── auth.js
├── utils/
│   ├── date.js
│   ├── string.js
│   └── validation.js
└── constants/
	├── config.js
	└── routes.js
```

---

## Consistent code formatting

Use tools to enforce consistent formatting.

```javascript
// ✅ Good - Consistent style (with Prettier/ESLint)

// Consistent spacing
const user = { age: 30, name: 'John' };
const items = [1, 2, 3];

// Consistent quotes (pick single or double, stick with it)
const message = 'Hello, world!';
const template = `Hello, ${name}!`;

// Consistent semicolons (use them or don't, be consistent)
const x = 1;
const y = 2;

// Consistent indentation (2 or 4 spaces, or tabs)
function example() {
	if (condition) {
		doSomething();
	}
}

// Consistent line length (usually 80-100 characters)
const longString = 
	'This is a very long string that would exceed the line length ' +
	'so we break it into multiple lines for readability.';

// Consistent object formatting
const config = {
	apiUrl: 'https://api.example.com',
	retries: 3,
	timeout: 5000
};

// Recommended .prettierrc
{
	"printWidth": 80,
	"semi": true,
	"singleQuote": true,
	"tabWidth": 2,
	"trailingComma": "es5"
}
```

---

## Avoid deep nesting

Flatten nested code for better readability.

```javascript
// ❌ Bad - Deep nesting
function processData(data) {
	if (data) {
		if (data.users) {
			if (data.users.length > 0) {
				for (const user of data.users) {
					if (user.active) {
						if (user.email) {
							if (isValidEmail(user.email)) {
								sendEmail(user.email);
							}
						}
					}
				}
			}
		}
	}
}

// ✅ Good - Early returns and extraction
function processData(data) {
	if (!data?.users?.length) {
		return;
	}
	
	const activeUsersWithValidEmail = data.users.filter(
		user => user.active && user.email && isValidEmail(user.email)
	);
	
	activeUsersWithValidEmail.forEach(user => sendEmail(user.email));
}

// ✅ Good - Guard clauses
async function updateUser(userId, updates) {
	if (!userId) {
		throw new Error('User ID is required');
	}
	
	if (!updates || Object.keys(updates).length === 0) {
		throw new Error('No updates provided');
	}
	
	const user = await getUser(userId);
	
	if (!user) {
		throw new NotFoundError('User not found');
	}
	
	if (!user.canBeEdited) {
		throw new Error('User cannot be edited');
	}
	
	// Main logic at the end, not nested
	return await saveUser(userId, updates);
}

// ✅ Good - Extract nested logic into functions
function getEligibleUsers(users) {
	return users.filter(isEligible);
}

function isEligible(user) {
	return user.active && user.verified && user.age >= 18;
}

function processEligibleUsers(users) {
	const eligible = getEligibleUsers(users);

	return eligible.map(processUser);
}
```

---

## Write comments that explain why

Comments should explain why, not what. Code shows what.

```javascript
// ❌ Bad - Comments explain what (obvious from code)
// Increment counter by 1
counter++;

// Loop through users
for (const user of users) {
	// Check if user is active
	if (user.active) {
		// Add user to list
		activeUsers.push(user);
	}
}

// ✅ Good - Comments explain why
// Counter must not exceed MAX_RETRIES to prevent infinite loops
if (counter >= MAX_RETRIES) {
	throw new Error('Max retries exceeded');
}

/*
We process users in reverse order because newer users 
have higher priority for notifications
*/
const sortedUsers = [...users].sort((a, b) => b.createdAt - a.createdAt);

// ✅ Good - Comments explain non-obvious decisions
/*
Using setTimeout instead of setInterval because the callback duration
varies and we want consistent gaps between executions
*/
function poll() {
	fetchData().finally(() => {
		setTimeout(poll, POLL_INTERVAL);
	});
}

// ✅ Good - Comments explain business rules
function calculateDiscount(order) {
	/*
	Business rule: Orders over $100 get 10% off,
	but discount cannot exceed $50 per company policy
	*/
	const discountPercent = order.total > 100 ? 0.10 : 0;
	const discount = order.total * discountPercent;
	const maxDiscount = 50;
	
	return Math.min(discount, maxDiscount);
}

// ✅ Good - TODO comments with context
/*
TODO(john): Refactor this once the new API is deployed (Q2 2024)
Current implementation uses deprecated endpoint
*/

// ✅ Good - Warning comments
/*
WARNING: This function modifies the database directly.
Do not call in production without proper authorization checks.
*/
function dangerousReset() {
	// ...
}

// ✅ Good - Comments for complex regex
/*
Matches email addresses: name@domain.tld
Captures: (1) local part, (2) domain, (3) TLD
*/
const emailRegex = /^([^@]+)@([^.]+)\.([a-z]{2,})$/i;
```

---

## Summary

| Principle | Description |
|-----------|-------------|
| Self-documenting | Clear naming eliminates need for comments |
| KISS | Choose simplest working solution |
| DRY | Extract repeated logic into reusable parts |
| Single responsibility | One function = one job |
| No magic values | Use named constants |
| Meaningful booleans | Extract complex conditions to variables |
| Short functions | Break large functions into smaller ones |
| Consistent formatting | Use automated tools (Prettier, ESLint) |
| Avoid nesting | Use early returns and extraction |
| Why, not what | Comments explain reasoning, not mechanics |
