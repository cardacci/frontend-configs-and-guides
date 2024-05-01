#### Indexed access types

Indexed access types in TypeScript allow you to access the type of a property in an object by its key. This is particularly useful when you have a union of keys and want to get the corresponding types of those keys in another type.

Let's break it down with examples:

Suppose we have an object type representing a person's information:

```typescript
type Person = {
	age: number;
	city: string;
	name: string;
};

```

Now, if we want to get the type of the ```age``` property from ```Person```, we can use indexed access types like this:

```typescript
type AgeType = Person['age']; // This will be 'number'.
```

Here, ```Person['age']``` accesses the type of the ```age``` property in the ```Person``` type, which is ```number```.

Indexed access types become more powerful when dealing with union types. For example, suppose we have a union of property names:

```typescript
type PersonKeys = 'age' | 'city' | 'name';
```

Now, if we want to get the types of all properties defined in ```PersonKeys```, we can use indexed access types like this:

```typescript
type PersonTypes = {
    [T in PersonKeys]: Person[T];
};

/*
This will be equivalent to:
type PersonTypes = {
	age: number;
	city: string;
	name: string;
};
*/
```

Here, ```[T in PersonKeys]``` iterates over each key in ```PersonKeys```, and ```Person[T]``` fetches the type of the corresponding property in ```Person```. As a result, ```PersonTypes``` will be a type that mirrors the structure of ```Person```, but with the types of properties specified by ```PersonKeys```.

This feature is incredibly useful when working with mapped types, generics, and other advanced type transformations in TypeScript. It allows for more flexible and dynamic type definitions, especially in scenarios where the structure of types needs to be inferred from other types or values.
