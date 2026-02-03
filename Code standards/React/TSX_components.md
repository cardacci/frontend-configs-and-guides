# TSX Components Structure Guide

> **Purpose:** This file defines the recommended order and organization for TSX
> components in React with TypeScript. It serves as both an **AI coding guideline**
> and a **developer reference** for writing and reviewing components consistently
> across the project.
>
> **Where this file lives:** See the [Project Setup](#project-setup) section at the bottom.

---

## Why This Structure Exists

React components can grow quickly and become hard to navigate. A consistent internal
structure means that any developer (or AI) reading the file can find what they need
predictably: types at the top, state in the middle, rendering at the bottom.
This order follows a "declare before you use" principle — everything a section depends
on is always defined above it.

---

## Alphabetical Ordering Rule

Everything that can be sorted alphabetically **must** be sorted alphabetically within
its block. This applies to:

- Enum values
- Interface and type properties
- Destructured props
- useState declarations
- useRef declarations
- useCallback declarations
- useEffect blocks (sorted by their primary dependency or action name)
- Function declarations
- Import specifiers within each import group
- Export lists

The only exception is when a declaration **depends** on another one defined in the same
block. In that case, the dependency comes first regardless of alphabetical order. This
is rare inside a single section — the structure is designed so that dependencies live
in earlier sections.

---

## Rule: TypeScript Definitions Must Live OUTSIDE the Component

**✅ DO: Define types, interfaces, and enums at module scope (above the component)**

```tsx
// ✅ Correct: defined outside, reusable and stable across renders
interface UserProps {
	name: string;
}

const User = ({ name }: UserProps) => <div>{name}</div>;
```

**❌ DON'T: Define types inside the component function body**

```tsx
// ❌ Wrong: not exportable, pollutes component scope
const User = () => {
	interface UserProps {
		// Never do this
		name: string;
	}

	return <div />;
};
```

Defining types outside gives you reusability across components, cleaner separation of
concerns, better IDE support and IntelliSense, and the ability to export them for
testing and reuse.

---

## Component Structure (Full Template)

This is the canonical order. Every section header below is a comment block you drop
into your `.tsx` file. Sections you don't need can be omitted entirely.

```tsx
/* ===== Imports ===== */
// 1. External libraries (React, third-party packages)
// 2. Redux or state management
// 3. Custom hooks
// 4. Child components
// 5. Types and interfaces (from other files)
// 6. Utils, constants, and assets
// → Within each group, sort import specifiers alphabetically

/* ===== Constants & Enums ===== */
// Enums first (they feed into types below), values sorted alphabetically
// Module-level constants sorted alphabetically (API URLs, limits, config objects, default values)

/* ===== Types & Interfaces ===== */
// Type aliases (can reference enums and constants above), properties sorted alphabetically
// Interfaces sorted alphabetically, properties sorted alphabetically
// Props interface always last (it references the types above)
// Generic types

/* ===== Component Function ===== */
// Function signature + props destructuring (alphabetical) with default values

/* ===== Redux ===== */
// useDispatch
// useSelector calls (sorted alphabetically by the state key they access)

/* ===== Hooks ===== */
// Custom hooks sorted alphabetically (e.g., useAuth, useDebounce, useFetch)

/* ===== State ===== */
// useState declarations sorted alphabetically by state variable name

/* ===== Refs ===== */
// useRef declarations sorted alphabetically

/* ===== Memos ===== */
// useMemo sorted alphabetically by variable name — derived or expensive computed values

/* ===== Derived Values ===== */
// Simple const declarations sorted alphabetically
// (no hooks — just plain const declarations computed from state/props)

/* ===== Functions ===== */
// Pure helper functions sorted alphabetically
// get[Value] for functions that return data
// render[Content] for functions that return JSX fragments

/* ===== Callbacks ===== */
// useCallback sorted alphabetically by function name
// Naming convention: handle[Action] for event handlers

/* ===== Effects ===== */
// useEffect blocks sorted alphabetically by their primary action or dependency

/* ===== JSX Return ===== */
// The component's return statement
// JSX props sorted alphabetically on each element

/* ===== Exports ===== */
// export default ComponentName
// export type { ... } sorted alphabetically
// export { ... } sorted alphabetically
```

---

## Section-by-Section Explanation

### Imports

Grouped by origin: external libraries first, then internal. Within each group, import
specifiers are sorted alphabetically. This makes it immediately clear what the component
depends on from outside the project vs. what comes from within.

### Constants & Enums

Enums go first because types and interfaces (in the next section) may reference them.
Enum values are sorted alphabetically. Constants like `DEFAULT_PAGE_SIZE` or
`MAX_RETRIES` belong here if they are reused across the file or exported, and are
sorted alphabetically among themselves. Keep configuration objects (e.g., API endpoint
maps) here as well, with their internal keys sorted alphabetically.

### Types & Interfaces

All TypeScript definitions for this file. Properties within each type or interface are
sorted alphabetically. Interfaces themselves are sorted alphabetically, with one
exception: the **Props interface is always last** — it acts as the "entry point" of the
component and often references the other types above it.

### Component Function

Just the function signature and props destructuring. Destructured props are sorted
alphabetically. Default prop values go here too. Everything else that follows is the
body of the component.

### Redux

`useDispatch` first, then selectors (`useSelector`) sorted alphabetically by the state key they read.
Keep them together because they represent the component's contract with
the global store.

### Hooks

Custom hooks that encapsulate logic, sorted alphabetically (e.g., `useAuth()`,
`useDebounce()`). These often return state or callbacks that the sections below will
consume.

### State

All `useState` declarations, sorted alphabetically by state variable name. Group
related state together and consider whether multiple pieces of state should be a single
`useReducer` instead.

### Refs

`useRef` declarations sorted alphabetically for DOM references, timers, or mutable
values that don't trigger re-renders.

### Memos

`useMemo` sorted alphabetically by variable name. Only use this when the computation is
genuinely costly or when the result is passed as a prop to a memoized child.

### Derived Values

Simple `const` declarations sorted alphabetically that derive a value from state or
props without needing `useMemo`. For example: `const fullName = \`${firstName} ${lastName}\`;`.
This is the right place for anything that's computed but doesn't need memoization.

### Functions

Pure, non-memoized helper functions sorted alphabetically. Use `get[Value]` naming for
functions that return data, and `render[Content]` naming for functions that return JSX
fragments.

### Callbacks

Memoized event handlers wrapped in `useCallback`, sorted alphabetically. Use
`handle[Action]` naming (e.g., `handleRowClick`, `handleSubmit`). These go after plain
functions because they may call them.

### Effects

`useEffect` blocks sorted alphabetically by their primary action or dependency name.
Place them last (before the return) because they run after render and often depend on
state, callbacks, or refs defined above.

### JSX Return

The component's `return (...)` statement. Props on each JSX element are sorted
alphabetically. If the JSX is complex, break it into `render[Name]` functions defined
in the Functions section above.

### Exports

Default export first, then named type exports (alphabetical), then named value exports
(alphabetical). Exporting types here makes them available for other components without
polluting the module's runtime exports.

---

## Naming Conventions

| Pattern           | Used For                                  | Example                          |
| ----------------- | ----------------------------------------- | -------------------------------- |
| `get[Value]`      | Functions that compute and return a value | `getFilteredUsers()`             |
| `handle[Action]`  | Event handler callbacks                   | `handleRowClick`, `handleSubmit` |
| `has[Condition]`  | Boolean derived values (presence)         | `hasErrors`, `hasResults`        |
| `is[Condition]`   | Boolean derived values (state)            | `isLoading`, `isSelected`        |
| `on[Action]`      | Props that receive callback functions     | `onChange`, `onUserSelect`       |
| `render[Content]` | Functions that return a JSX fragment      | `renderUserCard(user)`           |

---

## Example: Basic Component

A straightforward component that demonstrates the core structure without Redux or
custom hooks. Every block follows alphabetical ordering.

```tsx
/* ===== Imports ===== */
import React, { useCallback, useState } from 'react';

/* ===== Constants & Enums ===== */
enum Status {
	ERROR = 'error',
	IDLE = 'idle',
	LOADING = 'loading',
	SUCCESS = 'success',
}

const MAX_ITEMS = 100;

/* ===== Types & Interfaces ===== */
interface User {
	email: string;
	id: number;
	name: string;
	status: Status;
}

interface UserListProps {
	initialUsers: User[];
	onUserSelect: (user: User) => void;
}

/* ===== Component Function ===== */
const UserList = ({ initialUsers, onUserSelect }: UserListProps) => {
	/* ===== State ===== */
	const [selectedId, setSelectedId] = useState<number | null>(null);

	/* ===== Derived Values ===== */
	const activeUsers = initialUsers.filter((user) => user.status !== Status.ERROR);
	const hasUsers = activeUsers.length > 0;

	/* ===== Functions ===== */
	function renderUserCard(user: User): JSX.Element {
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
		(user: User) => {
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
};

/* ===== Exports ===== */
export default UserList;
export type { User, UserListProps };
export { Status };
```

---

## Example: Complex Component

A more realistic component that uses Redux, custom hooks, refs, memos, effects, and
multiple function types — showing how all sections work together with full alphabetical
ordering applied.

```tsx
/* ===== Imports ===== */
import React, { useCallback, useEffect, useMemo, useRef, useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { useDebounce } from '@/hooks/useDebounce';
import { UserCard } from '@/components/UserCard';
import type { RootState } from '@/store/types';
import { fetchUsers, selectUser } from '@/store/slices/usersSlice';
import { sortByName } from '@/utils/sorting';
import { DEBOUNCE_MS, MAX_VISIBLE_USERS } from '@/constants/config';

/* ===== Constants & Enums ===== */
enum SortDirection {
	ASC = 'asc',
	DESC = 'desc',
}

const PLACEHOLDER_TEXT = 'Search users...';

/* ===== Types & Interfaces ===== */
type SortField = 'createdAt' | 'email' | 'name';

interface FilterOptions {
	search: string;
	sortDirection: SortDirection;
	sortField: SortField;
}

interface UserListPageProps {
	defaultFilter?: Partial<FilterOptions>;
	onSelectionChange?: (userId: number | null) => void;
}

/* ===== Component Function ===== */
const UserListPage = ({ defaultFilter = {}, onSelectionChange }: UserListPageProps) => {
	/* ===== Redux ===== */
	const dispatch = useDispatch();
	const isLoading = useSelector((state: RootState) => state.users.isLoading);
	const users = useSelector((state: RootState) => state.users.list);

	/* ===== Hooks ===== */
	const debouncedSearch = useDebounce(DEBOUNCE_MS);

	/* ===== State ===== */
	const [search, setSearch] = useState(defaultFilter.search ?? '');
	const [selectedId, setSelectedId] = useState<number | null>(null);
	const [sortDirection, setSortDirection] = useState<SortDirection>(defaultFilter.sortDirection ?? SortDirection.ASC);
	const [sortField, setSortField] = useState<SortField>(defaultFilter.sortField ?? 'name');

	/* ===== Refs ===== */
	const listContainerRef = useRef<HTMLDivElement>(null);
	const searchInputRef = useRef<HTMLInputElement>(null);

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
	function getEmptyMessage(): string {
		return search.length > 0 ? `No users match "${search}".` : 'No users available yet.';
	}

	function renderUserCard(user: { email: string; id: number; name: string }): JSX.Element {
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
		(e: React.ChangeEvent<HTMLInputElement>) => {
			const value = e.target.value;

			debouncedSearch(value);
			setSearch(value);
		},
		[debouncedSearch]
	);

	const handleSelect = useCallback(
		(userId: number) => {
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
};

/* ===== Exports ===== */
export default UserListPage;
export type { FilterOptions, SortField, UserListPageProps };
export { SortDirection };
```

---

## Quick Reference Card

Use this as a checklist when writing or reviewing a component:

```
✅ Are all types/interfaces defined OUTSIDE the component?
✅ Is the Props interface the LAST interface in the Types section?
✅ Are imports grouped: external → internal hooks → components → types → utils?
✅ Are import specifiers sorted alphabetically within each group?
✅ Do enums come before the types that use them? Are enum values alphabetical?
✅ Are interface/type properties sorted alphabetically?
✅ Are destructured props sorted alphabetically?
✅ Are useState, useRef, useCallback, and useMemo sorted alphabetically?
✅ Are useEffect deps arrays sorted alphabetically?
✅ Are JSX props sorted alphabetically on each element?
✅ Are export lists sorted alphabetically?
✅ Is state declared before derived values that depend on it?
✅ Are effects placed after callbacks and functions they depend on?
✅ Are function names following: get[Value], render[Content], handle[Action]?
```
