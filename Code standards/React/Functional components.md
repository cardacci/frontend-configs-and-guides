In this article, a specific order and organization for a functional component in React are proposed.


```javascript
Function start...

Destructuring of component properties.

/* ===== Redux ===== */

/* ===== Hooks ===== */

/* ===== State ===== */

/* ===== Refs ===== */

/* ===== Dispatchs ===== */

/* ===== Memos ===== */

/* ===== Requests ===== */

/* ===== Variables ===== */

// COMPONENT'S OWN METHODS

/* ===== Callbacks ===== */

/* ===== Effects ===== */

/* ===== Custom Hooks ===== */

// COMPONENT'S RETURN
```

#### Method nomenclature

The form "get[VALUE_TO_BE_RETURNED]" is used for methods that return values.
Example: Texts, arrays, numbers, objects, etc.

```jsx
function getValue() {
	var value;
	...

	return value;
}
```

The form "render[CONTENT_TO_SHOW]" is used for methods that return HTML/JSX content.

```jsx
function renderElement() {
	return (
		<div>
			Content
		</div>
	);
}
```

#### Example of a component code

```jsx
import {
    useEffect, useCallback, useMemo, useRef, useState
} from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { useGlobalContext } from 'custom-hooks/index';
import { usePortfolioTypes } from 'custom-hooks/redux-hooks';
import { defaultValuesForNumberInString } from 'input-states';

function ExampleComponent(props) {
	const { name, var1, var2 } = props;

	/* ===== Redux ===== */
	const crossReducer = useSelector(state => state.cross);
	const userReducer= useSelector(state => state.user);
	const { reCaptchaToken } = crossReducer;
	const { bankAccountTypes, currencies, permissionsByRole, personDocumentTypes } = userReducer;

	/* ===== Hooks ===== */
	const { isMobile } = useGlobalContext();
	const portfolioTypesHook = usePortfolioTypes();
	// The hooks are ordered by their name and not by the constant, in this case useGlobalContext is positioned before usePortfolioTypes.

	/* ===== State ===== */
	const [count, setCount] = useState(0);
	/* ===== State: Inputs ===== */
	// This section contains the states corresponding to Inputs to be sent in a service.
	const [amount, setAmount] = useState(defaultValuesForNumberInString);
	/* ===== State: Form data ===== */
	// In this section are the states related to the behavior of the form (if it exists).
	const [entityProgressType, setEntityProgressType] = useState(null);
	const [errorMsg, setErrorMsg] = useState(null);

	/* ===== Refs ===== */
	const nameRef = useRef(null);

	/* ===== Dispatchs ===== */
	const dispatch = useDispatch();
	const onGotoPage = page => dispatch({ type: 'GOTO_PAGE', page });

	/* ===== Memos ===== */
	const autoComplete = useMemo(() => {
		let value = 'off';

		if (currentPassword || newPassword) {
			value = newPassword ? 'new-password' : 'current-password';
		}

		return value;
	}, [currentPassword, newPassword]);

	/* ===== Variables ===== */
	const const1 = 1989;
	const const2 = 1993;

	// MÉTODOS (*)
	function method1() {
		const auxA = var1;
	}

	function method2() {
		const auxA = var1;
		const auxB = var2;
	}

	function getValue() {
		var value;
		...
		return value;
	}

	function renderElement() {
		return (
			<div>
				{getValue()}
			</div>
		)
	}


	/* ===== Callbacks ===== */
	const handleKeyDown = useCallback(event => {
		if (count.length > 0) {
			console.log('You already clicked before');
		}
	}, [count]);

	/* ===== Effects ===== */
	useEffect(() => {
		document.title = `You clicked ${count} times`;
	});

	useEffect(() => {
		window.addEventListener('keydown', handleKeyDown, { capture: false });

		return () => {
			window.removeEventListener('keydown', handleKeyDown, { capture: false });
		};
	}, [handleKeyDown]);

	// JSX (*)
	return (
		<div>
			{renderElement()}

			<p>
				Hello {name}!
			</p>

			<p>
				You clicked {count} times
			</p>

			<button onClick={() => setCount(count + 1)}>
				Click me
			</button>
		</div>
	);
}
```