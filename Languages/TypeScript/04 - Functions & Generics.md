#### Functions

In this section, we will see how to type functions.

```typescript
interface Contact {
	clone(name: string): Contact
	id: number;
	name: string;
}

function clone(source: Contact): Contact {
	return Object.apply({}, source);
}

const a: Contact = {
	id: 123,
	name:'Homer Simpson'
};
const b = clone(a)
```

#### Generics


Generics in TypeScript are a powerful feature that allows writing flexible and reusable code when working with different data types. In simple terms, generics enable writing functions, classes, and types that can handle various data types without specifying them beforehand.

The key to generics is the ability to parameterize types. This means you can define a function or class with a type parameter that represents a specific type but can be used with any type the developer wants to provide. This provides greater flexibility and code reuse, as the same function or class can adapt to different data types.

For example, consider a function that takes an array and returns the first element of that array. Instead of specifying a specific type for the array elements, you can use a generic type so that the function can work with any type of array:

```typescript
function getFirstElement<T>(array: T[]): T | undefined {
    return array.length > 0 ? array[0] : undefined;
}

const numbers: number[] = [1, 2, 3, 4, 5];
const firstNumber: number | undefined = getFirstElement(numbers);

const words: string[] = ["Hello", "World"];
const firstWord: string | undefined = getFirstElement(words);
```

In this example, ```<T>``` is the generic type parameter. When we call the ```getFirstElement``` function, TypeScript infers the type of the T parameter based on the type of the array we pass as an argument. This allows us to use the same function with different types of arrays without having to rewrite the code.

In summary, generics in TypeScript are a powerful tool for writing flexible and reusable code by enabling the creation of functions, classes, and types that can work with different data types in a generic way. This helps improve code readability, maintainability, and scalability.
