# JSX Functional Components Structure Guide

> **Purpose:** This file defines the recommended order and organization for JSX
> functional components in React (without TypeScript). It is the JavaScript counterpart
> of `TSX_components.md` — both guides share the same structure, naming conventions,
> and alphabetical ordering rules. The only differences are the absence of types,
> interfaces, enums, and generics, and the use of `PropTypes` instead.

---

## How This Relates to the TSX Guide

The section order inside a component is identical in both guides. If you know one, you
know the other. The differences are limited to what JavaScript lacks compared to
TypeScript:

| TSX has | JSX equivalent |
|---|---|
| `enum` | Module-level `const` object (frozen) |
| `interface` / `type` | None — use `PropTypes` for runtime validation |
| Props interface | `ComponentName.propTypes` (defined outside the component) |
| `export type { ... }` | Not applicable |

Everything else — section order, naming conventions, callback structure, effect
placement — is exactly the same. Both guides also share the same alphabetical ordering
rule, which is defined once in each file and applies globally.

---

## Alphabetical Ordering Rule

Everything that can be sorted alphabetically **must** be sorted alphabetically within
its block. This applies to:

- Keys inside frozen constant objects
- Destructured props
- useState declarations
- useRef declarations
- useCallback declarations
- useEffect blocks (sorted by their primary action or dependency name)
- Function declarations
- Import specifiers within each import group
- PropTypes keys
- Export statements

The only exception is when a declaration **depends** on another one defined in the same
block. In that case, the dependency comes first regardless of alphabetical order.

---

## Component Structure (Full Template)

This is the canonical order. Every section header is a comment block you drop into your
`.jsx` file. Sections you don't need can be omitted entirely.

```jsx
/* ===== Imports ===== */
// 1. External libraries (React, third-party packages)
// 2. Redux or state management
// 3. Custom hooks
// 4. Child components
// 5. Utils, constants, and assets

/* ===== Constants ===== */
// Frozen objects that act as enums (see Enum Pattern below)
// Module-level constants, default values, and configuration objects

/* ===== Component Function ===== */
// Function signature + props destructuring with default values

/* ===== Redux ===== */
// useDispatch
// useSelector calls

/* ===== Hooks ===== */
// Custom hooks (e.g., useAuth, useDebounce, useFetch)

/* ===== State ===== */
// useState declarations

/* ===== Refs ===== */
// useRef declarations

/* ===== Memos ===== */
// useMemo — derived or expensive computed values

/* ===== Derived Values ===== */
// Simple const declarations computed from state/props (no hooks)

/* ===== Functions ===== */
// Pure helper functions (not memoized, no side effects)
// get[Value] for functions that return data
// render[Content] for functions that return JSX fragments

/* ===== Callbacks ===== */
// useCallback — event handlers and memoized functions
// Naming convention: handle[Action]

/* ===== Effects ===== */
// useEffect — side effects, subscriptions, data fetching

/* ===== JSX Return ===== */
// The component's return statement

/* ===== PropTypes ===== */
// ComponentName.defaultProps — default values for optional props
// ComponentName.propTypes — runtime type validation

/* ===== Exports ===== */
// export default ComponentName
```

---

## Section-by-Section Explanation

### Imports

Grouped by origin: external libraries first, then internal. This makes it immediately
clear what the component depends on from outside the project vs. what comes from within.

### Constants

Module-level constants that live outside the component. No enums exist in JavaScript,
so use frozen objects as a replacement when you need a fixed set of string values (see
the Enum Pattern section below). Constants like `DEFAULT_PAGE_SIZE` or `MAX_RETRIES`
belong here.

### Component Function

The function signature and props destructuring. Default values for props go here as
well, as an alternative to `defaultProps` when the component is simple. Everything else
that follows is the body of the component.

### Redux

`useDispatch` first, then selectors (`useSelector`). Keep them together — they
represent the component's contract with the global store. Destructure what you need
directly from the selector, do not create intermediate variables for the reducer object.

### Hooks

Custom hooks that encapsulate logic (e.g., `useAuth()`, `useDebounce()`). These are
ordered by hook name, not by the variable that receives the result. For example,
`useAuth()` comes before `useDebounce()` regardless of what you name the returned value.

### State

All `useState` declarations. If the component has a large form with many input states,
you may add sub-section comments like `// --- Inputs ---` and `// --- Form behavior ---`
to group related state, but alphabetical order must still be maintained within each
sub-section.

### Refs

`useRef` for DOM references, timers, or mutable values that don't trigger re-renders.

### Memos

`useMemo` for expensive computations. Only use this when the computation is genuinely
costly or when the result is passed as a prop to a memoized child.

### Derived Values

Simple `const` declarations that derive a value from state or props without needing
`useMemo`. This is where simple computed booleans like `hasResults` or concatenated
strings like `fullName` belong.

### Functions

Pure, non-memoized helper functions. Use `get[Value]` naming for functions that return
data, and `render[Content]` naming for functions that return JSX fragments.

### Callbacks

Memoized event handlers wrapped in `useCallback`. Use `handle[Action]` naming (e.g.,
`handleRowClick`, `handleSubmit`). These go after plain functions because they may call
them.

### Effects

`useEffect` blocks. Place them last (before the return) because they run after render
and often depend on state, callbacks, or refs defined above. Every variable used inside
an effect must appear in its dependency array.

### JSX Return

The component's `return (...)` statement. If the JSX is complex, break it into
`render[Name]` functions defined in the Functions section above.

### PropTypes

This section lives **outside** the component function, after it closes. It is the JSX
equivalent of the Props interface in the TSX guide — it documents what the component
expects and validates it at runtime.

`defaultProps` comes first, then `propTypes`. Both are required: `defaultProps` for
every optional prop, `propTypes` for every prop the component receives.

### Exports

A single `export default ComponentName` at the bottom. Do not use inline export on the
function declaration (`export default function ...`) — keep the export explicit and
separated, consistent with the TSX guide.

---

## Enum Pattern

JavaScript has no `enum` keyword. When you need a fixed set of named string values
(the same use case as an enum in TSX), use a frozen object defined in the Constants
section:

```jsx
/* ===== Constants ===== */
const Status = Object.freeze({
	ERROR: 'error',
	IDLE: 'idle',
	LOADING: 'loading',
	SUCCESS: 'success',
});
```

Keys are sorted alphabetically, exactly like enum values in TSX. Using `Object.freeze`
prevents accidental mutation and makes the object behave like a true constant at runtime.

---

## Naming Conventions

Identical to the TSX guide:

| Pattern                            | Used For                                  | Example                          |
| ---------------------------------- | ----------------------------------------- | -------------------------------- |
| `get[Value]`                       | Functions that compute and return a value | `getFilteredUsers()`             |
| `handle[Action]`                   | Event handler callbacks                   | `handleRowClick`, `handleSubmit` |
| `has[Condition]`                   | Boolean derived values (presence)         | `hasErrors`, `hasResults`        |
| `is[Condition]`                    | Boolean derived values (state)            | `isLoading`, `isSelected`        |
| `on[Action]`                       | Props that receive callback functions     | `onChange`, `onUserSelect`       |
| `render[Content]`                  | Functions that return a JSX fragment      | `renderUserCard(user)`           |

---

## Example: Basic Component

A straightforward component without Redux or custom hooks, showing the core structure.

```jsx
/* ===== Imports ===== */
import PropTypes from 'prop-types';
import { useCallback, useState } from 'react';

/* ===== Constants ===== */
const Status = Object.freeze({
	ERROR: 'error',
	IDLE: 'idle',
	LOADING: 'loading',
	SUCCESS: 'success',
});

/* ===== Component Function ===== */
function UserList({ initialUsers, onUserSelect }) {
	/* ===== State ===== */
	const [selectedId, setSelectedId] = useState(null);

	/* ===== Derived Values ===== */
	const activeUsers = initialUsers.filter((user) => user.status !== Status.ERROR);
	const hasUsers = activeUsers.length > 0;

	/* ===== Functions ===== */
	function renderUserCard(user) {
		return (
			<div
				className={user.id === selectedId ? 'selected' : ''}
				key={user.id}
				onClick={() => handleSelect(user)}
			>
				<span>
					{user.email}
				</span>

				<strong>
					{user.name}
				</strong>
			</div>
		);
	}

	/* ===== Callbacks ===== */
	const handleSelect = useCallback(
		(user) => {
			onUserSelect(user);
			setSelectedId(user.id);
		},
		[onUserSelect]
	);

	/* ===== JSX Return ===== */
	return (
		<div className="user-list">
			{hasUsers ? activeUsers.map(renderUserCard) : <p>No users available.</p>}
		</div>
	);
}

/* ===== PropTypes ===== */
UserList.defaultProps = {};

UserList.propTypes = {
	initialUsers: PropTypes.arrayOf(
		PropTypes.shape({
			email: PropTypes.string.isRequired,
			id: PropTypes.number.isRequired,
			name: PropTypes.string.isRequired,
			status: PropTypes.oneOf(Object.values(Status)).isRequired,
		})
	).isRequired,
	onUserSelect: PropTypes.func.isRequired,
};

/* ===== Exports ===== */
export default UserList;
```

---

## Example: Complex Component

A realistic component using Redux, custom hooks, refs, memos, effects, and multiple
function types — showing how every section works together.

```jsx
/* ===== Imports ===== */
import PropTypes from 'prop-types';
import { useCallback, useEffect, useMemo, useRef, useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { useDebounce } from '@/hooks/useDebounce';
import { UserCard } from '@/components/UserCard';
import { fetchUsers, selectUser } from '@/store/slices/usersSlice';
import { sortByName } from '@/utils/sorting';
import { DEBOUNCE_MS, MAX_VISIBLE_USERS } from '@/constants/config';

/* ===== Constants ===== */
const SortDirection = Object.freeze({
	ASC: 'asc',
	DESC: 'desc',
});

const DEFAULT_SORT_FIELD = 'name';
const PLACEHOLDER_TEXT = 'Search users...';

/* ===== Component Function ===== */
function UserListPage({ defaultFilter = {}, onSelectionChange }) {
	/* ===== Redux ===== */
	const dispatch = useDispatch();
	const isLoading = useSelector((state) => state.users.isLoading);
	const users = useSelector((state) => state.users.list);

	/* ===== Hooks ===== */
	const debouncedSearch = useDebounce(DEBOUNCE_MS);

	/* ===== State ===== */
	const [search, setSearch] = useState(defaultFilter.search ?? '');
	const [selectedId, setSelectedId] = useState(null);
	const [sortDirection, setSortDirection] = useState(defaultFilter.sortDirection ?? SortDirection.ASC);
	const [sortField, setSortField] = useState(defaultFilter.sortField ?? DEFAULT_SORT_FIELD);

	/* ===== Refs ===== */
	const listContainerRef = useRef(null);
	const searchInputRef = useRef(null);

	/* ===== Memos ===== */
	const filteredAndSortedUsers = useMemo(() => {
		const filtered = users.filter((user) =>
			user.name.toLowerCase().includes(search.toLowerCase())
		);

		return sortByName(filtered, sortField, sortDirection).slice(0, MAX_VISIBLE_USERS);
	}, [search, sortDirection, sortField, users]);

	/* ===== Derived Values ===== */
	const hasResults = filteredAndSortedUsers.length > 0;
	const resultCount = filteredAndSortedUsers.length;

	/* ===== Functions ===== */
	function getEmptyMessage() {
		return search.length > 0 ? `No users match "${search}".` : 'No users available yet.';
	}

	function renderUserCard(user) {
		return (
			<UserCard
				isSelected={user.id === selectedId}
				key={user.id}
				onSelect={() => handleSelect(user.id)}
				user={user}
			/>
		);
	}

	/* ===== Callbacks ===== */
	const handleSearchChange = useCallback(
		(e) => {
			const value = e.target.value;

			debouncedSearch(value);
			setSearch(value);
		},
		[debouncedSearch]
	);

	const handleSelect = useCallback(
		(userId) => {
			dispatch(selectUser(userId));
			onSelectionChange?.(userId);
			setSelectedId(userId);
		},
		[dispatch, onSelectionChange]
	);

	const handleSortToggle = useCallback(() => {
		setSortDirection((prev) =>
			prev === SortDirection.ASC ? SortDirection.DESC : SortDirection.ASC
		);
	}, []);

	/* ===== Effects ===== */
	useEffect(() => {
		searchInputRef.current?.focus();
	}, []);

	useEffect(() => {
		dispatch(fetchUsers());
	}, [dispatch]);

	useEffect(() => {
		onSelectionChange?.(selectedId);
	}, [onSelectionChange, selectedId]);

	/* ===== JSX Return ===== */
	return (
		<div className="user-list-page">
			<div className="controls">
				<input
					onChange={handleSearchChange}
					placeholder={PLACEHOLDER_TEXT}
					ref={searchInputRef}
					type="text"
					value={search}
				/>

				<button onClick={handleSortToggle}>
					Sort: {sortField} ({sortDirection})
				</button>
			</div>

			<p className="result-count">
				{isLoading ? 'Loading...' : `${resultCount} result(s)`}
			</p>

			<div className="user-list" ref={listContainerRef}>
				{hasResults ? (
					filteredAndSortedUsers.map(renderUserCard)
				) : (
					<p>{getEmptyMessage()}</p>
				)}
			</div>
		</div>
	);
}

/* ===== PropTypes ===== */
UserListPage.defaultProps = {
	defaultFilter: {},
	onSelectionChange: undefined,
};

UserListPage.propTypes = {
	defaultFilter: PropTypes.shape({
		search: PropTypes.string,
		sortDirection: PropTypes.oneOf(Object.values(SortDirection)),
		sortField: PropTypes.oneOf(['createdAt', 'email', DEFAULT_SORT_FIELD]),
	}),
	onSelectionChange: PropTypes.func,
};

/* ===== Exports ===== */
export default UserListPage;
```

---

## Quick Reference Card

Use this as a checklist when writing or reviewing a component:

```
✅ Are imports grouped: external → internal hooks → components → utils?
✅ Are frozen constant objects used instead of enums?
✅ Are custom hooks ordered by hook name (not by variable name)?
✅ Does every useEffect dependency array include all variables used inside it?
✅ Is every optional prop covered in defaultProps?
✅ Is the export at the bottom, not inline on the function declaration?
✅ Are function names following: get[Value], render[Content], handle[Action]?
✅ Is everything that can be sorted, sorted alphabetically within its block?
```
