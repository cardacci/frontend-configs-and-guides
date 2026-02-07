# Async Programming Best Practices

## Table of Contents

- [Prefer async/await over Promises](#prefer-asyncawait-over-promises)
- [Handle errors properly](#handle-errors-properly)
- [Avoid callback hell](#avoid-callback-hell)
- [Use Promise.all for parallel operations](#use-promiseall-for-parallel-operations)
- [Use Promise.allSettled when appropriate](#use-promiseallsettled-when-appropriate)
- [Avoid mixing async patterns](#avoid-mixing-async-patterns)
- [Handle loading and error states](#handle-loading-and-error-states)
- [Cancel unnecessary requests](#cancel-unnecessary-requests)
- [Debounce and throttle async operations](#debounce-and-throttle-async-operations)
- [Avoid async in constructors](#avoid-async-in-constructors)

---

## Prefer async/await over Promises

Async/await makes asynchronous code read like synchronous code.

```javascript
// ❌ Bad - Promise chains
function fetchUserData(userId) {
	return fetch(`/api/users/${userId}`)
		.then(response => response.json())
		.then(user => {
			return fetch(`/api/posts?userId=${user.id}`)
				.then(response => response.json())
				.then(posts => {
					return { user, posts };
				});
		});
}

// ✅ Good - async/await
async function fetchUserData(userId) {
	const userResponse = await fetch(`/api/users/${userId}`);
	const user = await userResponse.json();
	
	const postsResponse = await fetch(`/api/posts?userId=${user.id}`);
	const posts = await postsResponse.json();
	
	return { user, posts };
}

// ✅ Good - Clean and readable
async function processOrder(orderId) {
	const order = await getOrder(orderId);
	const inventory = await checkInventory(order.items);
	const payment = await processPayment(order);
	const shipment = await createShipment(order);
	
	return { order, inventory, payment, shipment };
}
```

---

## Handle errors properly

Always handle errors in async operations to prevent unhandled rejections.

```javascript
// ❌ Bad - No error handling
async function fetchData() {
	const response = await fetch('/api/data');

	return response.json();
}

// ❌ Bad - Empty catch
async function fetchData() {
	try {
		const response = await fetch('/api/data');

		return response.json();
	} catch (error) {
		// Swallowed error - bad!
	}
}

// ✅ Good - Proper error handling with try/catch
async function fetchData() {
	try {
		const response = await fetch('/api/data');
		
		if (!response.ok) {
			throw new Error(`HTTP error! status: ${response.status}`);
		}
		
		return await response.json();
	} catch (error) {
		console.error('Failed to fetch data:', error);

		throw error; // Re-throw or handle appropriately
	}
}

// ✅ Good - Centralized error handling
async function apiRequest(url, options = {}) {
	try {
		const response = await fetch(url, options);
		
		if (!response.ok) {
			const error = new Error(`API Error: ${response.status}`);

			error.status = response.status;
			error.response = response;

			throw error;
		}
		
		return await response.json();
	} catch (error) {
		if (error.name === 'AbortError') {
			console.log('Request was cancelled');

			return null;
		}

		throw error;
	}
}

// ✅ Good - Error handling at the calling level
async function loadUserProfile() {
	try {
		const user = await fetchUser();
		const posts = await fetchPosts(user.id);

		return { user, posts };
	} catch (error) {
		if (error.status === 404) {
			return { error: 'User not found' };
		}

		if (error.status === 401) {
			redirectToLogin();

			return null;
		}

		return { error: 'Something went wrong' };
	}
}
```

---

## Avoid callback hell

Transform nested callbacks into flat, readable async code.

```javascript
// ❌ Bad - Callback hell
function processUser(userId, callback) {
	getUser(userId, (err, user) => {
		if (err) {
			callback(err);

			return;
		}

		getPosts(user.id, (err, posts) => {
			if (err) {
				callback(err);

				return;
			}

			getComments(posts[0].id, (err, comments) => {
				if (err) {
					callback(err);

					return;
				}

				callback(null, { user, posts, comments });
			});
		});
	});
}

// ✅ Good - Promisify callbacks
function getUserAsync(userId) {
	return new Promise((resolve, reject) => {
		getUser(userId, (err, user) => {
			if (err) {
				reject(err);
			} else {
				resolve(user);
			}
		});
	});
}

// ✅ Good - Use async/await
async function processUser(userId) {
	const user = await getUserAsync(userId);
	const posts = await getPostsAsync(user.id);
	const comments = await getCommentsAsync(posts[0].id);
	
	return { user, posts, comments };
}

// ✅ Good - Use util.promisify (Node.js)
const { promisify } = require('util');

const getUserAsync = promisify(getUser);
```

---

## Use Promise.all for parallel operations

When operations don't depend on each other, run them in parallel.

```javascript
// ❌ Bad - Sequential when parallel is possible
async function loadDashboard(userId) {
	const user = await fetchUser(userId);
	const posts = await fetchPosts(userId);    // Waits for user
	const notifications = await fetchNotifications(userId); // Waits for posts
	
	return { user, posts, notifications };
}
// Total time: fetchUser + fetchPosts + fetchNotifications

// ✅ Good - Parallel execution
async function loadDashboard(userId) {
	const [user, posts, notifications] = await Promise.all([
		fetchUser(userId),
		fetchPosts(userId),
		fetchNotifications(userId),
	]);
	
	return { user, posts, notifications };
}
// Total time: max(fetchUser, fetchPosts, fetchNotifications)

// ✅ Good - Mix of parallel and sequential
async function loadUserDashboard(userId) {
	// First, get user (required for other calls)
	const user = await fetchUser(userId);
	
	// Then, fetch related data in parallel
	const [posts, followers, recommendations] = await Promise.all([
		fetchPosts(user.id),
		fetchFollowers(user.id),
		fetchRecommendations(user.preferences),
	]);
	
	return { user, posts, followers, recommendations };
}
```

---

## Use Promise.allSettled when appropriate

`Promise.allSettled` waits for all promises regardless of success/failure.

```javascript
// ❌ Bad - Promise.all fails fast
async function fetchAllData() {
	try {
		// If any fails, all results are lost
		const [users, posts, comments] = await Promise.all([
			fetchUsers(),
			fetchPosts(),    // If this fails...
			fetchComments(),
		]);

		return { users, posts, comments };
	} catch (error) {
		// We get no partial results
		return null;
	}
}

// ✅ Good - Promise.allSettled for resilience
async function fetchAllData() {
	const results = await Promise.allSettled([
		fetchUsers(),
		fetchPosts(),
		fetchComments(),
	]);
	
	const [usersResult, postsResult, commentsResult] = results;
	
	return {
		users: usersResult.status === 'fulfilled' ? usersResult.value : [],
		posts: postsResult.status === 'fulfilled' ? postsResult.value : [],
		comments: commentsResult.status === 'fulfilled' ? commentsResult.value : [],
		errors: results
			.filter(r => r.status === 'rejected')
			.map(r => r.reason),
	};
}

// ✅ Good - Helper function for allSettled
function getSettledValue(result, defaultValue = null) {
	return result.status === 'fulfilled' ? result.value : defaultValue;
}

async function loadPage() {
	const results = await Promise.allSettled([
		fetchHeader(),
		fetchContent(),
		fetchSidebar(),
	]);
	
	return {
		header: getSettledValue(results[0], defaultHeader),
		content: getSettledValue(results[1], defaultContent),
		sidebar: getSettledValue(results[2], defaultSidebar),
	};
}
```

---

## Avoid mixing async patterns

Be consistent with your async approach throughout the codebase.

```javascript
// ❌ Bad - Mixing patterns
async function getData() {
	const result = await fetch('/api/data')
		.then(res => res.json())  // Mixing await and .then()
		.catch(err => console.error(err));
	
	return result;
}

// ❌ Bad - Unnecessary Promise wrapper
async function fetchUser(id) {
	return new Promise(async (resolve, reject) => {  // Anti-pattern!
		try {
			const user = await fetch(`/api/users/${id}`);

			resolve(user.json());
		} catch (error) {
			reject(error);
		}
	});
}

// ✅ Good - Consistent async/await
async function getData() {
	try {
		const response = await fetch('/api/data');

		return await response.json();
	} catch (error) {
		console.error(error);

		throw error;
	}
}

// ✅ Good - async functions already return Promises
async function fetchUser(id) {
	const response = await fetch(`/api/users/${id}`);

	return response.json();
}
```

---

## Handle loading and error states

Always track loading and error states for better UX.

```javascript
// ✅ Good - State management for async operations
function useAsyncData(fetchFn) {
	const [data, setData] = useState(null);
	const [error, setError] = useState(null);
	const [loading, setLoading] = useState(false);
	
	const execute = async (...args) => {
		setLoading(true);
		setError(null);
		
		try {
			const result = await fetchFn(...args);

			setData(result);

			return result;
		} catch (err) {
			setError(err.message);

			throw err;
		} finally {
			setLoading(false);
		}
	};
	
	return { data, error, execute, loading };
}

// ✅ Good - Async operation wrapper
async function withLoadingState(asyncFn, setLoading, setError) {
	setLoading(true);
	setError(null);
	
	try {
		return await asyncFn();
	} catch (error) {
		setError(error.message);
		throw error;
	} finally {
		setLoading(false);
	}
}

// ✅ Good - Return object with status
async function fetchWithStatus(url) {
	try {
		const response = await fetch(url);
		const data = await response.json();

		return { success: true, data, error: null };
	} catch (error) {
		return { success: false, data: null, error: error.message };
	}
}
```

---

## Cancel unnecessary requests

Cancel pending requests when they're no longer needed.

```javascript
// ✅ Good - AbortController for fetch
async function fetchWithCancel(url) {
	const controller = new AbortController();
	
	const fetchPromise = fetch(url, {
		signal: controller.signal,
	});
	
	return {
		promise: fetchPromise,
		cancel: () => controller.abort(),
	};
}

// ✅ Good - Cancel on component unmount (React)
function useDataFetching(url) {
	const [data, setData] = useState(null);
	
	useEffect(() => {
		const controller = new AbortController();
		
		async function fetchData() {
			try {
				const response = await fetch(url, {
					signal: controller.signal,
				});
				const json = await response.json();

				setData(json);
			} catch (error) {
				if (error.name !== 'AbortError') {
					console.error('Fetch error:', error);
				}
			}
		}
		
		fetchData();
		
		return () => controller.abort(); // Cleanup
	}, [url]);
	
	return data;
}

// ✅ Good - Cancel previous request on new request
let currentController = null;

async function searchWithCancel(query) {
	// Cancel previous request
	if (currentController) {
		currentController.abort();
	}
	
	currentController = new AbortController();
	
	try {
		const response = await fetch(`/api/search?q=${query}`, {
			signal: currentController.signal,
		});

		return await response.json();
	} catch (error) {
		if (error.name === 'AbortError') {
			return null; // Cancelled, return nothing
		}

		throw error;
	}
}
```

---

## Debounce and throttle async operations

Prevent excessive API calls with debouncing and throttling.

```javascript
// ✅ Good - Debounce function
function debounce(fn, delay) {
	let timeoutId;
	
	return function (...args) {
		clearTimeout(timeoutId);
		timeoutId = setTimeout(() => fn.apply(this, args), delay);
	};
}

// Usage
const debouncedSearch = debounce(async (query) => {
	const results = await searchAPI(query);

	displayResults(results);
}, 300);

input.addEventListener('input', (e) => {
	debouncedSearch(e.target.value);
});

// ✅ Good - Debounce with async support
function debounceAsync(fn, delay) {
	let timeoutId;
	let pendingPromise = null;
	
	return function (...args) {
		clearTimeout(timeoutId);
		
		return new Promise((resolve, reject) => {
			timeoutId = setTimeout(async () => {
				try {
					const result = await fn.apply(this, args);

					resolve(result);
				} catch (error) {
					reject(error);
				}
			}, delay);
		});
	};
}

// ✅ Good - Throttle function
function throttle(fn, limit) {
	let inThrottle;
	
	return function (...args) {
		if (!inThrottle) {
			fn.apply(this, args);
			inThrottle = true;
			setTimeout(() => (inThrottle = false), limit);
		}
	};
}

// Usage - Limit scroll handler
const throttledScroll = throttle(async () => {
	if (isNearBottom()) {
		await loadMoreItems();
	}
}, 200);

window.addEventListener('scroll', throttledScroll);
```

---

## Avoid async in constructors

Constructors cannot be async. Use factory patterns instead.

```javascript
// ❌ Bad - Can't use async in constructor
class User {
	constructor(id) {
		// This doesn't work as expected!
		this.data = await fetchUser(id); // SyntaxError
	}
}

// ❌ Bad - Async without await in constructor
class User {
	constructor(id) {
		this.ready = this.init(id); // Promise not awaited
	}
	
	async init(id) {
		this.data = await fetchUser(id);
	}
}

// ✅ Good - Factory pattern
class User {
	constructor(data) {
		this.data = data;
	}
	
	static async create(id) {
		const data = await fetchUser(id);

		return new User(data);
	}
}

// Usage
const user = await User.create(123);

// ✅ Good - Initialization method
class DataService {
	constructor() {
		this.initialized = false;
		this.data = null;
	}
	
	async initialize() {
		if (this.initialized) {
			return;
		}
		
		this.data = await fetchData();
		this.initialized = true;
	}
	
	async getData() {
		await this.initialize();

		return this.data;
	}
}
```

---

## Summary

| Pattern | Description |
|---------|-------------|
| async/await | Prefer over raw Promises for readability |
| try/catch | Always handle errors properly |
| Promise.all | Run independent operations in parallel |
| Promise.allSettled | When you need results regardless of failures |
| AbortController | Cancel unnecessary requests |
| Debounce/Throttle | Limit frequency of async operations |
| Factory pattern | Avoid async in constructors |
| Loading states | Track loading, data, and error states |
