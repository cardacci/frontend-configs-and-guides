# TSX Components Standards

In this article, a specific order and organization for a TSX component in React with TypeScript are proposed, including interfaces, types, enums, constants and other TypeScript features.

## Component Structure

```typescript
/* ===== Imports ===== */
// Third-party libraries
// React and React-related imports
// Redux/State management imports
// Custom hooks
// Components
// Types and interfaces
// Utils and constants

/* ===== Constants & Enums ===== */
// Enums (first, as they can be used in types and interfaces)
// Component-level constants
// Default values
// Configuration objects

/* ===== Types & Interfaces ===== */
// Type aliases (can reference enums and constants)
// Interfaces (can reference types, enums and constants)
// Generic types

Function start...

Destructuring of component properties with proper typing.

/* ===== Redux ===== */

/* ===== Hooks ===== */

/* ===== State ===== */

/* ===== Refs ===== */

/* ===== Dispatchs ===== */

/* ===== Memos ===== */

/* ===== Requests ===== */

/* ===== Constants & Variables ===== */

/* ===== Functions ===== */

/* ===== Callbacks ===== */

/* ===== Effects ===== */

/* ===== Custom Hooks ===== */

// COMPONENT'S RETURN
```

## TypeScript Features in Components

### 1. Constants & Enums (Defined First)
Constants and enums should be defined before types and interfaces since they can be referenced by them.

#### Enums
Enums define named constants for better code organization:

```typescript
enum UserStatus {
	ACTIVE = 'active',
	INACTIVE = 'inactive',
	PENDING = 'pending',
	SUSPENDED = 'suspended'
}

enum ApiEndpoints {
	USERS = '/api/users',
	POSTS = '/api/posts',
	COMMENTS = '/api/comments'
}
```

#### Constants
Component-level constants that can be used in types and interfaces:

```typescript
const MAX_NAME_LENGTH: number = 50;
const DEFAULT_USER_ROLE: UserRole = UserRole.USER; // References enum
const EMAIL_REGEX: RegExp = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

// Configuration objects using enums and constants
const STATUS_MESSAGES: Record<ComponentStatus, string> = {
	[ComponentStatus.IDLE]: 'Ready to start',
	[ComponentStatus.LOADING]: 'Processing...',
	[ComponentStatus.SUCCESS]: 'Operation completed successfully',
	[ComponentStatus.ERROR]: 'An error occurred'
};
```

### 2. Types
Type aliases create reusable type definitions and can reference previously defined enums and constants:

```typescript
type Status = 'loading' | 'success' | 'error';
type UserRole = 'admin' | 'user' | 'guest';
type EventHandler<T = HTMLElement> = (event: React.MouseEvent<T>) => void;
```

### 3. Interfaces
Interfaces define the shape of objects, component props, and state structures. They can reference types, enums, and constants:

```typescript
interface User {
	email: string;
	id: number;
	isActive: boolean;
	name: string;
	profile?: UserProfile; // Optional property
	role: UserRole; // References type/enum defined above
}

interface ComponentProps {
	children?: React.ReactNode;
	className?: string;
	maxNameLength?: typeof MAX_NAME_LENGTH; // References constant
	onUserUpdate: (user: User) => void;
	user: User;
}
```

### 4. Generics
Generic types provide reusability and type safety:

```typescript
interface ApiResponse<T> {
	data: T;
	message: string;
	success: boolean;
}

interface PaginatedResponse<T> extends ApiResponse<T[]> {
	pagination: {
		limit: number;
		page: number;
		total: number;
	};
}
```

### 5. Utility Types
TypeScript utility types for advanced type manipulation:

```typescript
type PartialUser = Partial<User>; // All properties optional
type RequiredUser = Required<User>; // All properties required
type UserEmail = Pick<User, 'email'>; // Only email property
type UserWithoutId = Omit<User, 'id'>; // All except id
type UserKeys = keyof User; // 'id' | 'name' | 'email' | 'isActive' | 'profile'
```

## Method Nomenclature

The form "get[VALUE_TO_BE_RETURNED]" is used for methods that return values.
Example: Texts, arrays, numbers, objects, etc.

```tsx
function getValue(): string {
	const value: string = "example";
	// ...

	return value;
}

function getUsers(): User[] {
	// ...

	return users;
}
```

The form "render[CONTENT_TO_SHOW]" is used for methods that return JSX content.

```tsx
function renderUserCard(user: User): JSX.Element {
	return (
		<div className='user-card'>
			<h3>
				{user.name}
			</h3>

			<p>
				{user.email}
			</p>
		</div>
	);
}
```

## Example of a TSX Component

```tsx
import React, {
	useEffect, 
	useCallback, 
	useMemo, 
	useRef, 
	useState,
	FC
} from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { useGlobalContext } from 'custom-hooks/index';
import { usePortfolioTypes } from 'custom-hooks/redux-hooks';
import { RootState } from 'store/types';
import { UserActions } from 'store/actions';

/* ===== Constants & Enums ===== */
enum ComponentStatus {
	ERROR = 'error'
	IDLE = 'idle',
	LOADING = 'loading',
	SUCCESS = 'success',
}

enum UserRole {
	ADMIN = 'admin',
	GUEST = 'guest'
	USER = 'user',
}

const DEFAULT_USER_ROLE: UserRole = UserRole.USER;
const MAX_NAME_LENGTH: number = 50;
const EMAIL_REGEX: RegExp = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

const STATUS_MESSAGES: Record<ComponentStatus, string> = {
	[ComponentStatus.IDLE]: 'Ready to start',
	[ComponentStatus.LOADING]: 'Processing...',
	[ComponentStatus.SUCCESS]: 'Operation completed successfully',
	[ComponentStatus.ERROR]: 'An error occurred'
};

/* ===== Types & Interfaces ===== */
type AlertType = 'error' | 'info' | 'success' | 'warning';

interface User {
	createdAt: Date;
	email: string;
	id: number;
	isActive: boolean;
	name: string;
	profile?: UserProfile;
	role: UserRole;
}

interface UserProfile {
	avatar: string;
	bio: string;
	preferences: UserPreferences;
}

interface UserPreferences {
	language: string;
	notifications: boolean;
	theme: 'light' | 'dark';
}

interface ComponentProps {
	children?: React.ReactNode;
	className?: string;
	initialUser?: User;
	isReadOnly?: boolean;
	maxUsers?: number;
	onUserCreate: (user: Omit<User, 'id' | 'createdAt'>) => Promise<User>;
	onUserDelete: (id: number) => Promise<void>;
	onUserUpdate: (id: number, updates: Partial<User>) => Promise<User>;
}

interface FormData {
	name: string;
	email: string;
	role: UserRole;
}

interface ValidationErrors {
	name?: string;
	email?: string;
	role?: string;
	general?: string;
}

interface ApiResponse<T> {
	data: T;
	message: string;
	success: boolean;
	timestamp: Date;
}

/* ===== Component ===== */
const ExampleTSXComponent: FC<ComponentProps> = ({
	children
	className = '',
	initialUser,
	isReadOnly = false,
	maxUsers = 100,
	onUserCreate,
	onUserDelete,
	onUserUpdate,
}) => {
	/* ===== Redux ===== */
	const dispatch = useDispatch();
	const { users, loading, error } = useSelector((state: RootState) => state.user);
	const { theme, permissions } = useSelector((state: RootState) => state.app);

	/* ===== Hooks ===== */
	const { isMobile, isTablet } = useGlobalContext();
	const portfolioTypesHook = usePortfolioTypes();

	/* ===== State ===== */
	const [status, setStatus] = useState<ComponentStatus>(ComponentStatus.IDLE);
	const [selectedUser, setSelectedUser] = useState<User | null>(initialUser || null);
	const [userList, setUserList] = useState<User[]>([]);

	/* ===== State: Form data ===== */
	const [formData, setFormData] = useState<FormData>(FORM_INITIAL_STATE);
	const [validationErrors, setValidationErrors] = useState<ValidationErrors>({});
	const [isFormValid, setIsFormValid] = useState<boolean>(false);
	const [showConfirmDialog, setShowConfirmDialog] = useState<boolean>(false);

	/* ===== State: UI State ===== */
	const [alertType, setAlertType] = useState<AlertType>('info');
	const [alertMessage, setAlertMessage] = useState<string>('');
	const [isAlertVisible, setIsAlertVisible] = useState<boolean>(false);

	/* ===== Refs ===== */
	const nameInputRef = useRef<HTMLInputElement>(null);
	const formRef = useRef<HTMLFormElement>(null);
	const userListRef = useRef<HTMLDivElement>(null);

  /* ===== Dispatches ===== */
  const handleSetUsers = useCallback((users: User[]) => {
	dispatch(UserActions.setUsers(users));
  }, [dispatch]);

  const handleSetLoading = useCallback((loading: boolean) => {
	dispatch(UserActions.setLoading(loading));
  }, [dispatch]);

  /* ===== Memos ===== */
  const filteredUsers = useMemo<User[]>(() => {
	return userList.filter(user => 
	  user.isActive && 
	  user.role !== UserRole.GUEST
	);
  }, [userList]);

  const userCount = useMemo<number>(() => {
	return filteredUsers.length;
  }, [filteredUsers]);

  const canAddMoreUsers = useMemo<boolean>(() => {
	return userCount < maxUsers && !isReadOnly;
  }, [userCount, maxUsers, isReadOnly]);

  const statusMessage = useMemo<string>(() => {
	return STATUS_MESSAGES[status];
  }, [status]);

  /* ===== Constants & Variables ===== */
  const componentClasses: string = `example-component ${className} ${theme}`.trim();
  const isFormDisabled: boolean = status === ComponentStatus.LOADING || isReadOnly;

  /* ===== Functions ===== */
  function validateForm(data: FormData): ValidationErrors {
	const errors: ValidationErrors = {};

	if (!data.name.trim()) {
	  errors.name = 'Name is required';
	} else if (data.name.length > MAX_NAME_LENGTH) {
	  errors.name = `Name must be ${MAX_NAME_LENGTH} characters or less`;
	}

	if (!data.email.trim()) {
	  errors.email = 'Email is required';
	} else if (!EMAIL_REGEX.test(data.email)) {
	  errors.email = 'Please enter a valid email address';
	}

	if (!Object.values(UserRole).includes(data.role)) {
	  errors.role = 'Please select a valid role';
	}

	return errors;
  }

  function resetForm(): void {
	setFormData(FORM_INITIAL_STATE);
	setValidationErrors({});
	setIsFormValid(false);
  }

  function showAlert(type: AlertType, message: string): void {
	setAlertType(type);
	setAlertMessage(message);
	setIsAlertVisible(true);
  }

  function getUserById(id: number): User | undefined {
	return userList.find(user => user.id === id);
  }

  function getUsersByRole(role: UserRole): User[] {
	return userList.filter(user => user.role === role);
  }

  function formatUserDisplayName(user: User): string {
	return `${user.name} (${user.email})`;
  }

  function isUserAdmin(user: User): boolean {
	return user.role === UserRole.ADMIN;
  }

  /* ===== Render Functions ===== */
  function renderUserCard(user: User): JSX.Element {
	return (
	  <div key={user.id} className="user-card">
		<div className="user-info">
		  <h3>{user.name}</h3>
		  <p>{user.email}</p>
		  <span className={`role-badge role-${user.role}`}>
			{user.role.toUpperCase()}
		  </span>
		</div>
		<div className="user-actions">
		  {!isReadOnly && (
			<>
			  <button 
				type="button"
				onClick={() => handleEditUser(user)}
				disabled={isFormDisabled}
			  >
				Edit
			  </button>
			  <button 
				type="button"
				onClick={() => handleDeleteUser(user.id)}
				disabled={isFormDisabled}
				className="danger"
			  >
				Delete
			  </button>
			</>
		  )}
		</div>
	  </div>
	);
  }

  function renderUserList(): JSX.Element {
	if (filteredUsers.length === 0) {
	  return (
		<div className="empty-state">
		  <p>No users found</p>
		</div>
	  );
	}

	return (
	  <div className="user-list" ref={userListRef}>
		{filteredUsers.map(renderUserCard)}
	  </div>
	);
  }

  function renderForm(): JSX.Element {
	return (
	  <form ref={formRef} onSubmit={handleSubmit} className="user-form">
		<div className="form-group">
		  <label htmlFor="name">Name</label>
		  <input
			ref={nameInputRef}
			id="name"
			type="text"
			value={formData.name}
			onChange={handleNameChange}
			disabled={isFormDisabled}
			className={validationErrors.name ? 'error' : ''}
		  />
		  {validationErrors.name && (
			<span className="error-message">{validationErrors.name}</span>
		  )}
		</div>

		<div className="form-group">
		  <label htmlFor="email">Email</label>
		  <input
			id="email"
			type="email"
			value={formData.email}
			onChange={handleEmailChange}
			disabled={isFormDisabled}
			className={validationErrors.email ? 'error' : ''}
		  />
		  {validationErrors.email && (
			<span className="error-message">{validationErrors.email}</span>
		  )}
		</div>

		<div className="form-group">
		  <label htmlFor="role">Role</label>
		  <select
			id="role"
			value={formData.role}
			onChange={handleRoleChange}
			disabled={isFormDisabled}
			className={validationErrors.role ? 'error' : ''}
		  >
			{Object.values(UserRole).map(role => (
			  <option key={role} value={role}>
				{role.charAt(0).toUpperCase() + role.slice(1)}
			  </option>
			))}
		  </select>
		  {validationErrors.role && (
			<span className="error-message">{validationErrors.role}</span>
		  )}
		</div>

		<div className="form-actions">
		  <button
			type="submit"
			disabled={!isFormValid || isFormDisabled || !canAddMoreUsers}
		  >
			{selectedUser ? 'Update User' : 'Create User'}
		  </button>
		  <button
			type="button"
			onClick={resetForm}
			disabled={isFormDisabled}
		  >
			Reset
		  </button>
		</div>
	  </form>
	);
  }

  function renderAlert(): JSX.Element | null {
	if (!isAlertVisible) return null;

	return (
	  <div className={`alert alert-${alertType}`}>
		<span>{alertMessage}</span>
		<button 
		  type="button"
		  onClick={() => setIsAlertVisible(false)}
		  className="alert-close"
		>
		  ×
		</button>
	  </div>
	);
  }

  /* ===== Callbacks ===== */
  const handleNameChange = useCallback((event: React.ChangeEvent<HTMLInputElement>): void => {
	const newFormData: FormData = { ...formData, name: event.target.value };
	setFormData(newFormData);
	
	const errors = validateForm(newFormData);
	setValidationErrors(errors);
	setIsFormValid(Object.keys(errors).length === 0);
  }, [formData]);

  const handleEmailChange = useCallback((event: React.ChangeEvent<HTMLInputElement>): void => {
	const newFormData: FormData = { ...formData, email: event.target.value };
	setFormData(newFormData);
	
	const errors = validateForm(newFormData);
	setValidationErrors(errors);
	setIsFormValid(Object.keys(errors).length === 0);
  }, [formData]);

  const handleRoleChange = useCallback((event: React.ChangeEvent<HTMLSelectElement>): void => {
	const newFormData: FormData = { ...formData, role: event.target.value as UserRole };
	setFormData(newFormData);
	
	const errors = validateForm(newFormData);
	setValidationErrors(errors);
	setIsFormValid(Object.keys(errors).length === 0);
  }, [formData]);

  const handleSubmit = useCallback(async (event: React.FormEvent<HTMLFormElement>): Promise<void> => {
	event.preventDefault();
	
	if (!isFormValid) return;

	setStatus(ComponentStatus.LOADING);

	try {
	  if (selectedUser) {
		// Update existing user
		const updatedUser = await onUserUpdate(selectedUser.id, formData);
		setUserList(prev => prev.map(user => 
		  user.id === selectedUser.id ? updatedUser : user
		));
		showAlert('success', 'User updated successfully');
	  } else {
		// Create new user
		const newUser = await onUserCreate(formData);
		setUserList(prev => [...prev, newUser]);
		showAlert('success', 'User created successfully');
	  }
	  
	  resetForm();
	  setSelectedUser(null);
	  setStatus(ComponentStatus.SUCCESS);
	} catch (error) {
	  console.error('Error saving user:', error);
	  showAlert('error', 'Failed to save user. Please try again.');
	  setStatus(ComponentStatus.ERROR);
	}
  }, [formData, isFormValid, selectedUser, onUserCreate, onUserUpdate]);

  const handleEditUser = useCallback((user: User): void => {
	setSelectedUser(user);
	setFormData({
	  name: user.name,
	  email: user.email,
	  role: user.role
	});
	
	// Focus on name input
	setTimeout(() => {
	  nameInputRef.current?.focus();
	}, 0);
  }, []);

  const handleDeleteUser = useCallback(async (userId: number): Promise<void> => {
	if (!window.confirm('Are you sure you want to delete this user?')) {
	  return;
	}

	setStatus(ComponentStatus.LOADING);

	try {
	  await onUserDelete(userId);
	  setUserList(prev => prev.filter(user => user.id !== userId));
	  
	  if (selectedUser?.id === userId) {
		resetForm();
		setSelectedUser(null);
	  }
	  
	  showAlert('success', 'User deleted successfully');
	  setStatus(ComponentStatus.SUCCESS);
	} catch (error) {
	  console.error('Error deleting user:', error);
	  showAlert('error', 'Failed to delete user. Please try again.');
	  setStatus(ComponentStatus.ERROR);
	}
  }, [selectedUser, onUserDelete]);

  const handleKeyDown = useCallback((event: KeyboardEvent): void => {
	if (event.key === 'Escape') {
	  resetForm();
	  setSelectedUser(null);
	}
  }, []);

  /* ===== Effects ===== */
  // Mount effect
  useEffect(() => {
	nameInputRef.current?.focus();
	setStatus(ComponentStatus.IDLE);
  }, []);

  // Keyboard listener effect
  useEffect(() => {
	window.addEventListener('keydown', handleKeyDown);

	return () => {
	  window.removeEventListener('keydown', handleKeyDown);
	};
  }, [handleKeyDown]);

  // Form validation effect
  useEffect(() => {
	const errors = validateForm(formData);
	setValidationErrors(errors);
	setIsFormValid(Object.keys(errors).length === 0);
  }, [formData]);

  // Auto-hide alert effect
  useEffect(() => {
	if (isAlertVisible) {
	  const timer = setTimeout(() => {
		setIsAlertVisible(false);
	  }, 5000);

	  return () => clearTimeout(timer);
	}
  }, [isAlertVisible]);

  /* ===== Custom Hooks ===== */
  // Any custom hooks would go here

  // COMPONENT'S RETURN
  return (
	<div className={componentClasses}>
	  {renderAlert()}
	  
	  <header className="component-header">
		<h2>User Management</h2>
		<div className="component-status">
		  Status: <span className={`status-${status}`}>{statusMessage}</span>
		</div>
		<div className="user-count">
		  Users: {userCount} / {maxUsers}
		</div>
	  </header>

	  <main className="component-content">
		{!isReadOnly && (
		  <section className="form-section">
			<h3>{selectedUser ? 'Edit User' : 'Create New User'}</h3>
			{renderForm()}
		  </section>
		)}

		<section className="users-section">
		  <h3>Users ({userCount})</h3>
		  {renderUserList()}
		</section>
	  </main>

	  {children && (
		<footer className="component-footer">
		  {children}
		</footer>
	  )}
	</div>
  );
};

// Default props using TypeScript default parameters (modern approach)
// No need for separate defaultProps declaration

export default ExampleTSXComponent;

// Type exports for external use
export type {
  ComponentProps as ExampleComponentProps,
  User,
  UserRole,
  FormData as UserFormData,
  ValidationErrors as UserValidationErrors
};

export { UserStatus, UserRole };
```

## Key TypeScript Features Explained

### 1. **Strict Typing**
- All variables, function parameters, and return types are explicitly typed
- Props interface defines exact shape of component properties
- State variables have specific types

### 2. **Enums for Constants**
- `ComponentStatus` enum for component states
- `UserRole` enum for user permissions
- Better autocomplete and type safety

### 3. **Interface Composition**
- Interfaces can extend other interfaces
- Optional properties with `?`
- Readonly properties for immutable data

### 4. **Generic Types**
- `ApiResponse<T>` for type-safe API responses
- `FC<ComponentProps>` for functional component typing
- Reusable across different data types

### 5. **Utility Types**
- `Partial<User>` for optional updates
- `Omit<User, 'id'>` for creation without ID
- `Record<Key, Value>` for object mapping

### 6. **Type Guards & Assertions**
- Runtime type checking
- Type narrowing with conditional logic
- Safe type casting when needed

### 7. **Event Typing**
- Specific React event types like `React.ChangeEvent<HTMLInputElement>`
- Proper keyboard event handling
- Type-safe event handlers

This structure provides maximum type safety while maintaining clean, readable code that leverages all of TypeScript's powerful features in a React component context.
