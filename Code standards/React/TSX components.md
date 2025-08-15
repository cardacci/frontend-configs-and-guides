# TSX Components Structure Guide

This file defines the recommended order and organization for TSX components in React with TypeScript.

## Important: TypeScript Definitions Placement

**✅ RECOMMENDED: Define types, interfaces, and enums OUTSIDE the component function**

- Better performance (not recreated on each render)
- Reusability across components
- Cleaner separation of concerns
- Better IDE support and IntelliSense
- Can be exported for testing and reuse

**❌ AVOID: Defining types inside the component function**

## Component Structure

```typescript
/* ===== Imports ===== */
// External libraries (React, Redux, etc.)
// Custom hooks
// Components
// Types and interfaces
// Utils and constants

/* ===== Constants & Enums ===== */
// Enums (first, as they can be used in types and interfaces)
// Component constants
// Default values
// Configuration objects

/* ===== Types & Interfaces ===== */
// Type aliases (can reference enums and constants)
// Interfaces (can reference types, enums and constants)
// Generic types

/* ===== Component Function ===== */
// Component function start
// Props destructuring with typing

/* ===== Redux ===== */
// Selectors and dispatchers

/* ===== Hooks ===== */
// Custom hooks

/* ===== State ===== */
// Local component state

/* ===== Refs ===== */
// References

/* ===== Memos ===== */
// Memoized values

/* ===== Constants & Variables ===== */
// Local component variables and constants

/* ===== Functions ===== */
// Component functions

/* ===== Callbacks ===== */
// Memoized functions

/* ===== Effects ===== */
// Side effects

/* ===== Custom Hooks ===== */
// Custom hooks usage (if applicable)

// Component JSX
```

## Basic Example

```tsx
import React, { useState, useEffect, useCallback } from 'react';

/* ===== Constants & Enums ===== */
const MAX_ITEMS = 100;

enum Status {
	IDLE = 'idle',
	LOADING = 'loading',
	SUCCESS = 'success'
}

/* ===== Types & Interfaces ===== */
interface User {
	email: string;
	id: number;
	name: string;
	status: Status;
}

interface UserListProps {
	onUserSelect: (user: User) => void;
	users: User[];
}

const UserList: React.FC<UserListProps> = (props) => {
	const { users, onUserSelect } = props;

	/* ===== State ===== */
	const [selectedId, setSelectedId] = useState<number | null>(null);

	/* ===== Callbacks ===== */
	const handleSelect = useCallback((user: User) => {
		setSelectedId(user.id);
		onUserSelect(user);
	}, [onUserSelect]);

	/* ===== Effects ===== */
	useEffect(() => {
		// Component mount logic
	}, []);

	return (
		<div>
			{users.map(user => (
				<div
					key={user.id}
					onClick={() => handleSelect(user)}
				>
					{user.name}
				</div>
			))}
		</div>
	);
};

export default UserList;

// Export types for reuse in other components
export type { User, UserListProps };
export { Status };
```

## Method Nomenclature

- **get[VALUE]**: For methods that return values
- **render[CONTENT]**: For methods that return JSX

```tsx
function getFilteredUsers(): User[] {
	// ...
	return filteredUsers;
}

function renderUserCard(user: User): JSX.Element {
	return <div>{user.name}</div>;
}
```
