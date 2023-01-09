# Understanding Nullish Coalescing in Typescript


## What Nullish Coalescing is all about

Nullish coalescing is an operator introduced in [TypeScript version 3.7](https://github.com/microsoft/TypeScript-Website/blob/v2/packages/documentation/copy/en/release-notes/TypeScript%203.7.md#nullish-coalescing) that allows developers to specify a default value for `null` or `undefined`. This can be especially useful when working with strict null checking, as it allows you to differentiate between an explicitly set null or undefined value and a missing value that should be set to a default.

The nullish coalescing operator, represented by `??` in Typescript (and Javascript as well), works similarly to the logical OR operator `||`, but only returns the right-hand side operand if the left-hand side operand is strictly `null` or `undefined`. Which is quite different from the logical OR operator which will return the right-hand side operand for any falsy value, including `0`, `''`, and `false`.

## Let's dive in shall we?

Here is an example of nullish coalescing in action:

```ts
const value: number | undefined = someFunction();

const otherValue = value ?? 0; // would default to 0 if value is undefined

console.log(otherValue);

```

In the above example, assuming you were expecting a number from a function that could possibly return `undefined` using nullish coalescing, we where able to default the value of the variable `otherValue` to `0` .

Nullish coalescing can be especially useful when working with optional parameters or destructured objects. let's look at another example:

```ts
function greet(name: string | null | undefined) { 
console.log(`Hello, ${name ?? 'there'}!`);
}  

greet('Joe'); // Output: "Hello, Joe!" 
greet(null); // Output: "Hello, there!" 
greet(undefined); // Output: "Hello, there!"
```


```ts
const { status } = response.data

const approvalStatus = status ?? "PENDING";
```

## Comparing Nullish Coalescing to the Logical OR operator

To compare nullish coalescing to the logical OR operator in typescript, let's take a look at the code snippet below:

```ts
const emptyString = '';

const firstValue = emptyString ?? 0;

const secondValue = emptyString || 0;
```

if you didn't know about the difference between falsey values and nullish values in Javascript, you might be tempted to think that the variable `firstValue` would be `0`  just like the `secondValue`. It is important not to forget that falsey values are values that are considered false(in Javascript for example those are `0` , `''` , and `false` as mentioned earlier) while nullish values are either `null` or `undefined`.

```ts
console.log(firstValue); // output: ''

console.log(secondValue); // output: 0
```

## Wrapping it up

We have learned about nullish coalescing and how it compares to the Logical OR Operator in Typescript. In summary of it all, it's best to use nullish coalescing to set the default value if your use-case is dependent on the left-hand side operand being a nullish value not a falsey value (even though all nullish values are falsey). Hope you have learned something see you at the next one.


