# Error Handling Best Practices

## Table of Contents

- [Always use try-catch for risky operations](#always-use-try-catch-for-risky-operations)
- [Create custom error classes](#create-custom-error-classes)
- [Provide meaningful error messages](#provide-meaningful-error-messages)
- [Don't swallow errors](#dont-swallow-errors)
- [Use error boundaries in React](#use-error-boundaries-in-react)
- [Validate input early](#validate-input-early)
- [Log errors properly](#log-errors-properly)
- [Handle different error types](#handle-different-error-types)
- [Graceful degradation](#graceful-degradation)
- [Centralized error handling](#centralized-error-handling)

---

## Always use try-catch for risky operations

Wrap operations that might fail in try-catch blocks.

```javascript
// ❌ Bad - No error handling
function parseJSON(jsonString) {
	return JSON.parse(jsonString); // Throws on invalid JSON
}

// ❌ Bad - No error handling for async
async function fetchUser(id) {
	const response = await fetch(`/api/users/${id}`);

	return response.json();
}

// ✅ Good - Proper error handling
function parseJSON(jsonString) {
	try {
		return JSON.parse(jsonString);
	} catch (error) {
		console.error('Failed to parse JSON:', error.message);

		return null;
	}
}

// ✅ Good - Async error handling
async function fetchUser(id) {
	try {
		const response = await fetch(`/api/users/${id}`);
		
		if (!response.ok) {
			throw new Error(`HTTP ${response.status}: ${response.statusText}`);
		}
		
		return await response.json();
	} catch (error) {
		console.error(`Failed to fetch user ${id}:`, error);

		throw error;
	}
}

// ✅ Good - Handle specific risky operations
function processFile(file) {
	let content;
	
	try {
		content = readFile(file);
	} catch (error) {
		throw new Error(`Cannot read file: ${file}`);
	}
	
	let parsed;

	try {
		parsed = JSON.parse(content);
	} catch (error) {
		throw new Error(`Invalid JSON in file: ${file}`);
	}
	
	return parsed;
}
```

---

## Create custom error classes

Custom errors make error handling more precise and informative.

```javascript
// ✅ Good - Custom error classes
class AppError extends Error {
	constructor(message, statusCode = 500) {
		super(message);

		this.name = this.constructor.name;
		this.statusCode = statusCode;

		Error.captureStackTrace(this, this.constructor);
	}
}

class ValidationError extends AppError {
	constructor(message, field = null) {
		super(message, 400);
		this.field = field;
	}
}

class NotFoundError extends AppError {
	constructor(resource, id) {
		super(`${resource} with id ${id} not found`, 404);

		this.resource = resource;
		this.resourceId = id;
	}
}

class AuthenticationError extends AppError {
	constructor(message = 'Authentication required') {
		super(message, 401);
	}
}

class AuthorizationError extends AppError {
	constructor(message = 'Permission denied') {
		super(message, 403);
	}
}

// ✅ Good - Usage
async function getUser(id) {
	const user = await database.findUser(id);
	
	if (!user) {
		throw new NotFoundError('User', id);
	}
	
	return user;
}

function validateEmail(email) {
	if (!email) {
		throw new ValidationError('Email is required', 'email');
	}
	
	if (!email.includes('@')) {
		throw new ValidationError('Invalid email format', 'email');
	}
}

// ✅ Good - Handle specific error types
async function handleRequest(req, res) {
	try {
		const result = await processRequest(req);

		res.json(result);
	} catch (error) {
		if (error instanceof ValidationError) {
			res.status(400).json({ error: error.message, field: error.field });
		} else if (error instanceof NotFoundError) {
			res.status(404).json({ error: error.message });
		} else if (error instanceof AuthenticationError) {
			res.status(401).json({ error: error.message });
		} else {
			res.status(500).json({ error: 'Internal server error' });
		}
	}
}
```

---

## Provide meaningful error messages

Error messages should help identify and fix the problem.

```javascript
// ❌ Bad - Vague error messages
throw new Error('Error');
throw new Error('Something went wrong');
throw new Error('Invalid input');

// ✅ Good - Descriptive error messages
throw new Error('Failed to connect to database: connection timeout after 30s');
throw new Error(`User with email "${email}" already exists`);
throw new Error(`Invalid date format: expected YYYY-MM-DD, got "${input}"`);

// ✅ Good - Include context
function processOrder(order) {
	if (!order.id) {
		throw new Error('Order ID is required');
	}
	
	if (!order.items || order.items.length === 0) {
		throw new Error(`Order ${order.id} has no items`);
	}
	
	if (order.total < 0) {
		throw new Error(
			`Order ${order.id} has invalid total: ${order.total}. ` +
			`Total must be a positive number.`
		);
	}
}

// ✅ Good - Error with solution hints
function loadConfig(path) {
	try {
		return JSON.parse(fs.readFileSync(path));
	} catch (error) {
		if (error.code === 'ENOENT') {
			throw new Error(
				`Config file not found at "${path}". ` +
				`Create a config.json file or set CONFIG_PATH environment variable.`
			);
		}

		if (error instanceof SyntaxError) {
			throw new Error(
				`Invalid JSON in config file "${path}": ${error.message}. ` +
				`Check for missing commas or quotes.`
			);
		}

		throw error;
	}
}
```

---

## Don't swallow errors

Never catch errors without handling them properly.

```javascript
// ❌ Bad - Silently swallowing errors
try {
	doSomething();
} catch (error) {
	// Silent fail - very bad!
}

// ❌ Bad - Just logging without handling
try {
	await saveData();
} catch (error) {
	console.log(error); // User never knows something failed
}

// ✅ Good - Handle and inform
try {
	await saveData();
} catch (error) {
	console.error('Failed to save data:', error);
	showErrorToUser('Could not save your changes. Please try again.');

	throw error; // Re-throw if caller needs to know
}

// ✅ Good - Provide fallback
function getConfig() {
	try {
		return JSON.parse(localStorage.getItem('config'));
	} catch (error) {
		console.warn('Failed to load config, using defaults:', error);

		return getDefaultConfig();
	}
}

// ✅ Good - Log and rethrow with context
async function createUser(userData) {
	try {
		return await database.insert('users', userData);
	} catch (error) {
		console.error('Database error creating user:', {
			error: error.message,
			userData: { ...userData, password: '[REDACTED]' },
		});

		throw new Error('Failed to create user account');
	}
}
```

---

## Use error boundaries in React

Error boundaries catch JavaScript errors in component trees.

```javascript
// ✅ Good - Error boundary component
class ErrorBoundary extends React.Component {
	constructor(props) {
		super(props);

		this.state = { hasError: false, error: null };
	}

	static getDerivedStateFromError(error) {
		return { hasError: true, error };
	}

	componentDidCatch(error, errorInfo) {
		console.error('Error caught by boundary:', error);
		console.error('Component stack:', errorInfo.componentStack);
		
		// Log to error reporting service
		errorReportingService.log({
			error,
			componentStack: errorInfo.componentStack,
		});
	}

	render() {
		if (this.state.hasError) {
			return this.props.fallback || (
				<div className="error-fallback">
					<h2>
						Something went wrong
					</h2>

					<button onClick={() => this.setState({ hasError: false })}>
						Try again
					</button>
				</div>
			);
		}

		return this.props.children;
	}
}

// ✅ Good - Usage
function App() {
	return (
		<ErrorBoundary fallback={<ErrorPage />}>
			<Header />

			<ErrorBoundary fallback={<ContentError />}>
				<MainContent />
			</ErrorBoundary>

			<Footer />
		</ErrorBoundary>
	);
}

// ✅ Good - Reusable error boundary with reset
function ErrorFallback({ error, resetErrorBoundary }) {
	return (
		<div role="alert" className="error-container">
			<h2>
				Something went wrong
			</h2>

			<pre>
				{error.message}
			</pre>

			<button onClick={resetErrorBoundary}>
				Try again
			</button>
		</div>
	);
}
```

---

## Validate input early

Validate inputs at the start of functions to fail fast.

```javascript
// ❌ Bad - Validation scattered or missing
function processPayment(amount, cardNumber) {
	// Processing starts without validation
	const processor = getPaymentProcessor();

	// Error happens deep in the process
	return processor.charge(amount, cardNumber);
}

// ✅ Good - Early validation
function processPayment(amount, cardNumber) {
	// Validate immediately
	if (typeof amount !== 'number' || isNaN(amount)) {
		throw new ValidationError('Amount must be a valid number');
	}
	
	if (amount <= 0) {
		throw new ValidationError('Amount must be positive');
	}
	
	if (!cardNumber || typeof cardNumber !== 'string') {
		throw new ValidationError('Card number is required');
	}
	
	if (!/^\d{16}$/.test(cardNumber.replace(/\s/g, ''))) {
		throw new ValidationError('Invalid card number format');
	}
	
	// Now safe to proceed
	const processor = getPaymentProcessor();

	return processor.charge(amount, cardNumber);
}

// ✅ Good - Validation helper functions
function validateRequired(value, fieldName) {
	if (value === undefined || value === null || value === '') {
		throw new ValidationError(`${fieldName} is required`, fieldName);
	}
}

function validateEmail(email) {
	validateRequired(email, 'email');
	
	const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

	if (!emailRegex.test(email)) {
		throw new ValidationError('Invalid email format', 'email');
	}
}

function validateUser(user) {
	validateRequired(user.name, 'name');
	validateEmail(user.email);
	
	if (user.age !== undefined && (user.age < 0 || user.age > 150)) {
		throw new ValidationError('Invalid age', 'age');
	}
}
```

---

## Log errors properly

Implement structured logging for better debugging.

```javascript
// ❌ Bad - Basic console.log
try {
	doSomething();
} catch (error) {
	console.log(error);
}

// ✅ Good - Structured error logging
const logger = {
	error(message, context = {}) {
		console.error(JSON.stringify({
			level: 'error',
			message,
			timestamp: new Date().toISOString(),
			...context,
		}));
	},
	
	warn(message, context = {}) {
		console.warn(JSON.stringify({
			level: 'warn',
			message,
			timestamp: new Date().toISOString(),
			...context,
		}));
	},
};

// ✅ Good - Log with context
async function createOrder(orderData) {
	const requestId = generateRequestId();
	
	try {
		logger.info('Creating order', {
			requestId,
			userId: orderData.userId,
			itemCount: orderData.items.length,
		});
		
		const order = await database.createOrder(orderData);
		
		logger.info('Order created successfully', {
			requestId,
			orderId: order.id,
		});
		
		return order;
	} catch (error) {
		logger.error('Failed to create order', {
			requestId,
			userId: orderData.userId,
			error: error.message,
			stack: error.stack,
		});

		throw error;
	}
}

// ✅ Good - Redact sensitive data
function sanitizeForLogging(data) {
	const sensitiveFields = ['creditCard', 'password', 'ssn', 'token'];
	const sanitized = { ...data };
	
	for (const field of sensitiveFields) {
		if (sanitized[field]) {
			sanitized[field] = '[REDACTED]';
		}
	}
	
	return sanitized;
}

logger.error('Login failed', {
	user: sanitizeForLogging(userData),
	error: error.message,
});
```

---

## Handle different error types

Handle different error types appropriately.

```javascript
// ✅ Good - Type-specific error handling
async function fetchData(url) {
	try {
		const response = await fetch(url);

		return await response.json();
	} catch (error) {
		// Network errors
		if (error instanceof TypeError && error.message.includes('fetch')) {
			throw new Error('Network error: Please check your connection');
		}
		
		// Timeout errors
		if (error.name === 'AbortError') {
			throw new Error('Request timed out: Please try again');
		}
		
		// JSON parsing errors
		if (error instanceof SyntaxError) {
			throw new Error('Invalid response format from server');
		}
		
		// Re-throw unknown errors
		throw error;
	}
}

// ✅ Good - Handle HTTP status codes
async function apiRequest(url, options = {}) {
	const response = await fetch(url, options);
	
	if (response.ok) {
		return response.json();
	}
	
	// Handle specific status codes
	switch (response.status) {
		case 400:
			const errors = await response.json();

			throw new ValidationError('Invalid request', errors);
		
		case 401:
			throw new AuthenticationError('Please log in to continue');
		
		case 403:
			throw new AuthorizationError('You don\'t have permission');
		
		case 404:
			throw new NotFoundError('Resource not found');
		
		case 429:
			const retryAfter = response.headers.get('Retry-After');

			throw new RateLimitError(`Too many requests. Retry after ${retryAfter}s`);
		
		case 500:
		case 502:
		case 503:
			throw new ServerError('Server error. Please try again later.');
		
		default:
			throw new Error(`Unexpected error: ${response.status}`);
	}
}
```

---

## Graceful degradation

When errors occur, provide degraded functionality instead of complete failure.

```javascript
// ✅ Good - Fallback values
async function getUserPreferences(userId) {
	const defaultPreferences = {
		language: 'en',
		notifications: true,
		theme: 'light'
	};
	
	try {
		const preferences = await fetchPreferences(userId);

		return { ...defaultPreferences, ...preferences };
	} catch (error) {
		console.warn('Failed to load preferences, using defaults:', error);

		return defaultPreferences;
	}
}

// ✅ Good - Feature degradation
async function loadDashboard() {
	const [userData, analyticsData, recommendationsData] = 
		await Promise.allSettled([
			fetchUserData(),
			fetchAnalytics(),
			fetchRecommendations(),
		]);
	
	return {
		user: userData.status === 'fulfilled' 
			? userData.value 
			: getGuestUser(),
		
		analytics: analyticsData.status === 'fulfilled'
			? analyticsData.value
			: { available: false, message: 'Analytics temporarily unavailable' },
		
		recommendations: recommendationsData.status === 'fulfilled'
			? recommendationsData.value
			: [],
	};
}

// ✅ Good - Retry with fallback
async function fetchWithRetry(url, retries = 3) {
	for (let attempt = 1; attempt <= retries; attempt++) {
		try {
			return await fetch(url);
		} catch (error) {
			console.warn(`Attempt ${attempt} failed:`, error.message);
			
			if (attempt === retries) {
				throw new Error(`Failed after ${retries} attempts: ${error.message}`);
			}
			
			// Exponential backoff
			await new Promise(resolve => 
				setTimeout(resolve, Math.pow(2, attempt) * 1000)
			);
		}
	}
}
```

---

## Centralized error handling

Create a central place to handle all errors consistently.

```javascript
// ✅ Good - Global error handler
window.addEventListener('error', (event) => {
	console.error('Global error:', event.error);
	errorReportingService.log(event.error);
});

window.addEventListener('unhandledrejection', (event) => {
	console.error('Unhandled promise rejection:', event.reason);
	errorReportingService.log(event.reason);
});

// ✅ Good - Express error middleware
function errorHandler(err, req, res, next) {
	// Log error
	console.error({
		message: err.message,
		method: req.method,
		stack: err.stack,
		url: req.url,
		userId: req.user?.id
	});
	
	// Don't leak error details in production
	const isDev = process.env.NODE_ENV === 'development';
	
	if (err instanceof ValidationError) {
		return res.status(400).json({
			details: err.details,
			error: 'Validation Error',
			message: err.message
		});
	}
	
	if (err instanceof NotFoundError) {
		return res.status(404).json({
			error: 'Not Found',
			message: err.message
		});
	}
	
	// Generic error response
	res.status(err.statusCode || 500).json({
		error: 'Internal Server Error',
		message: isDev ? err.message : 'Something went wrong',
		stack: isDev ? err.stack : undefined
	});
}

// ✅ Good - Async handler wrapper
function asyncHandler(fn) {
	return (req, res, next) => {
		Promise.resolve(fn(req, res, next)).catch(next);
	};
}

// Usage
app.get('/users/:id', asyncHandler(async (req, res) => {
	const user = await getUser(req.params.id);

	res.json(user);
}));

app.use(errorHandler);
```

---

## Summary

| Practice | Description |
|----------|-------------|
| try-catch | Wrap risky operations |
| Custom errors | Create specific error classes |
| Meaningful messages | Include context and solutions |
| Don't swallow | Always handle or re-throw |
| Error boundaries | Catch React component errors |
| Early validation | Fail fast with clear messages |
| Structured logging | Log with context, redact sensitive data |
| Type handling | Handle different errors differently |
| Graceful degradation | Provide fallbacks when possible |
| Centralization | Handle errors in one place |
