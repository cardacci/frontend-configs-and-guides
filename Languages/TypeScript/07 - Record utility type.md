#### Record utility type

The ```Record``` utility type in TypeScript is used to create a new type with specified property keys and their corresponding value types. It's particularly useful when you want to define an object type with known keys and a consistent value type for each key.

Let's explain using an example:

Suppose we have a type representing the contact information of individuals, where each contact has a unique ID as a key and an associated Contact object as its value:

```typescript
interface Contact {
	email: string;
	name: string;
	phone: string;
}

type ContactsDatabase = Record<number, Contact>;
```

In this example, ```ContactsDatabase``` is defined using the ```Record``` utility type. It specifies that the keys of the object will be ```number``` (representing the contact ID) and the values will be of type ```Contact```.

Now, we can use ```ContactsDatabase``` to define an object that stores contact information:

```typescript
const contacts: ContactsDatabase = {
	1: {
		email: 'john.d@example.com',
		name: 'John Doe',
		phone: '2257558899'
	},
	2: {
		email: 'jane.s@example.com',
		name: 'Jane Smith',
		phone: '2235887466'
	}
};
```

Here, ```contacts``` is an object where each key is a contact ID (of type ```number```) and each value is a ```Contact``` object containing the name, email, and phone number.

Another use case for ```Record``` is when you want to ensure that an object has a specific set of keys with a default value for each key. For example:

```typescript
const defaults: Record<string, number> = {
	'depth': 50,
	'height': 200,
	'width': 100
};
```

In this case, ```defaults``` is an object where the keys are strings representing dimensions, and the values are default numeric values. The ```Record<string, number>``` type ensures that all keys are strings, and all values are numbers.

In summary, the ```Record``` utility type in TypeScript is a powerful tool for defining object types with specific keys and their corresponding value types, providing better type safety and expressiveness in your code.