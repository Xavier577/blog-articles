# Into Typescript's utility Types

Typescript offers a few useful features which enhance developer productivity working with types. Utility Types are one of these features, they are types that are available globally meaning they are in-built and work without you importing them from anywhere. These types can be used to manipulate existing types and create new types by utilizing generics and other type features such as conditional types.

In this article, we would be going over a few in-built utility types, and how to use them, and also we would be creating one to demonstrate how we can use them to create new types. If you want to learn more about in-built utility types that are not covered in this article, you can check out the [utility types section](https://www.typescriptlang.org/docs/handbook/utility-types.html) in type typescript documentation.

## Let's dive into some utility types

* `Partial<Type>`: We can use the `Partial` type to make all properties of a type optional which is the equivalent of making all the properties optional. Let's take a look at an example:
    

```ts
const todo: { 
	title?: string;
	description?: string;
} = {}; // this compiler won't ask you to add the properties
```

```ts
interface Todo {
	title: string;
	description: string;
}

const todo: Partial<Todo> = {}; // this compiler won't ask you to add the properties
```

the snippets above are equivalent to each other.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675108968878/eb5e802d-96f3-4d50-990c-911e755004bb.png align="center")

* `Omit<Type, Keys>`: We can use `Omit` to create a type from an interface by excluding some keys (which would exclude the property with that key).
    

```ts
interface Person {
	firstName: string;
	lastName: string;
	age: number;
	dateOfBirth: number;
};

type PersonWithoutDateOfBirth = Omit<Person, "dateOfBirth">;


const personWithoutDateOfBirth: PersonWithoutDateOfBirth = {
	firstName: "John",
	lastName: "Doe",
	age: 22
};
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675108752070/5201cdf8-d383-4d58-8cb6-18213acac8bf.png align="center")

This can be combined with other typescript features to create more advanced utility types (we would be creating one in a sec).

* `Record<Keys, Type>`: This is used to construct an object type whose properties keys are the first argument type of the generic (`Keys`) in this case and their types being the second argument type of the generic (`Type`). Meaning when the Object properties would have the same type we can use the `Record` utility type to set their types instead of defining them one by one. Let's see it in action:
    

```ts
interface Person {
	id: number;
	email: string;
};

type People = "daniel" | "solomon" |  "matt";

const people: Record<People, Person> = {
	daniel: { id: 1, email: "daniel@gmail.com" },
	solomon: { id: 2, email: "solomon@gmail.com"},
	matt: {id: 3, email: "matt@gmail.com" }
};
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675108506681/2e13b229-acb1-4032-ba60-644fd2d0ba84.png align="center")

* `Readonly<Type>`: This is used to set the `Type` to be readonly i.e you can't change the value of the property (typescript compiler would scream at you).
    

```ts
interface Person {
	id: number
	email: string;
};

const John: Readonly<Person> = {
	id: 68,
	email: "johdoe@gmail.com"
};

John.email = 1; // compiler error: Cannot assign to 'email' because it is a read-only property.
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675109360080/e960a067-25f8-4c86-bf05-775b0d04a87b.png align="center")

## Let's create our own utility Type

In some cases, you may need to create custom utility types that are not provided by TypeScript by default. This can be done by combining type features such as generics, condition typing, type aliases and so on.

The type we would be creating is a solution to a twitter post I came across.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675109704342/657a0ae8-d379-428a-aa71-9e387dbad56d.jpeg align="center")

so from this we are gonna be creating a utility type which would generate a type which is the odd property between two objects, hence the "Diff" or let's just say the difference between two object types. Let's get into the code snippet before I get to the breakdown of it.

```ts
type Diff<A,B> = A extends B ? Omit<A, keyof B> : Omit<B, keyof A>;

interface A  {
    a: string;
    b: string;
    c: string;
};

interface B {
    a: string;
    b: string;
};
   
type C = Diff<A,B>;

const c: C = {
	c: "c"
};
```

so here was my solution to it. It might look funny at first but it quite simple. Let's break it down step by step.

* First, we must remember that Utility types make use of generics so our two arguments of the generic `A` and `B` represent the two object types.
    
* Then we made use of conditional typing to check if `A` extends `B` which means A is derived type of `B` meaning it has all the properties of `B` .
    
* If `A` does extend `B` that would mean `A` has all the properties of `B` and a few new properties so we use `Omit` to remove all the properties of `A` that are in `B` (we used `Omit<A, keyof B>` to remove all the properties in `A` that has the same key as the property in `B`) leaving behind only the properties in `A` that are not in `B`.
    
* If `A` does not extend `B` it would mean that `B` has properties that are not present in `A` so we then use `Omit` to remove all the properties of `B` that are in `A` leaving behind only properties in `B` that are not in `A`.
    

That's just about it, it's not as complicated as it might look.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675110804555/cee8b6ae-a148-4ca5-81d2-34ec513f26cc.png align="center")

## Conclusion

Utility Types in TypeScript are a powerful feature that can be used to manipulate existing types and create new types in a type-safe manner. TypeScript provides a number of built-in utility types that can be used in different parts of your application, and it's also possible to create custom utility types when needed.

In conclusion, Utility Types can greatly improve the quality of your code, making it easier to write generic code and ensuring type safety. Until the next one :) .