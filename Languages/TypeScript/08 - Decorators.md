#### Decorators

Decorators in TypeScript are a special kind of declaration that can be attached to a class, method, accessor, property, or parameter. Decorators provide a way to add both annotations and a meta-programming syntax for class declarations and members. Decorators are an experimental feature and might change in future releases.

To enable decorators in TypeScript, you need to enable the ```experimentalDecorators``` compiler option in your ```tsconfig.json```:

```json
{
	"compilerOptions": {
		"experimentalDecorators": true
	}
}
```

##### Creating a Method Decorator

A method decorator is a function that takes three arguments: the target object, the name of the method, and the property descriptor of the method. Here’s an example of a simple method decorator:

```typescript
function Log(target: any, propertyName: string, descriptor: PropertyDescriptor) {
	const originalMethod = descriptor.value;

	descriptor.value = function (...args: any[]) {
		console.log(`Calling ${propertyName} with arguments:`, args);

		const result = originalMethod.apply(this, args);

		console.log(`Result:`, result);

		return result;
	};

	return descriptor;
}

class ExampleClass {
	@Log
	sayHello(name: string) {
		return `Hello, ${name}!`;
	}
}

const example = new ExampleClass();

example.sayHello('Gabriel');

// Output:
// Calling sayHello with arguments: ['Gabriel']
// Result: 'Hello, Gabriel!'
```

##### Creating Decorator Factories

A decorator factory is a function that returns a decorator. This is useful when you want to pass arguments to your decorator. Here’s an example:

```typescript
function LogPrefix(prefix: string) {
	return function (target: any, propertyName: string, descriptor: PropertyDescriptor) {
		const originalMethod = descriptor.value;

		descriptor.value = function (...args: any[]) {
			console.log(`${prefix} Calling ${propertyName} with arguments:`, args);

			const result = originalMethod.apply(this, args);

			console.log(`${prefix} Result:`, result);

			return result;
		};

		return descriptor;
	};
}

class ExampleClassWithPrefix {
	@LogPrefix('Prefix:')
	sayHello(name: string) {
		return `Hello, ${name}!`;
	}
}

const exampleWithPrefix = new ExampleClassWithPrefix();

exampleWithPrefix.sayHello('Gabriel');

// Output:
// Prefix: Calling sayHello with arguments: ['Gabriel']
// Prefix: Result: 'Hello, Gabriel!'
```

##### Creating a Class Decorator

A class decorator is a function that takes a single argument: the constructor of the class to be decorated. It can be used to modify or replace the class constructor. Here’s an example:

```typescript
function AddGreeting(target: Function) {
	target.prototype.greet = function () {
		console.log('Hello from the class decorator!');
	};
}

@AddGreeting
class Greeter {}

const greeter = new Greeter();

(greeter as any).greet();

// Output:
// Hello from the class decorator!
```

##### Creating a Property Decorator

A property decorator is a function that takes two arguments: the target object and the name of the property. Here’s an example:

```typescript
function ReadOnly(target: any, propertyName: string) {
	const descriptor: PropertyDescriptor = {
		get() {
			return 'This is a read-only property';
		},
		set() {
			console.log('Cannot set value to a read-only property');
		},
		enumerable: true,
		configurable: true
	};

	Object.defineProperty(target, propertyName, descriptor);
}

class ExampleWithReadOnly {
    @ReadOnly
    readOnlyProperty: string;
}

const exampleWithReadOnly = new ExampleWithReadOnly();

console.log(exampleWithReadOnly.readOnlyProperty);

// Output:
// This is a read-only property

exampleWithReadOnly.readOnlyProperty = 'New Value';
// Output:
// Cannot set value to a read-only property
```

##### Summary

Decorators in TypeScript provide a powerful way to add meta-programming capabilities to your code. They can be applied to classes, methods, properties, and parameters, and can be used to add functionality, logging, validation, and more. By creating decorator factories, you can further customize and reuse decorators across your application.
