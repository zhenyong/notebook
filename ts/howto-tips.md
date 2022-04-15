## Giving specify the tuple type explicitly and derive the union from it
```js
const ALL_SUITS = ['hearts', 'diamonds', 'spades', 'clubs'] as const;
type SuitTuple = typeof ALL_SUITS; // readonly ['hearts', 'diamonds', 'spades', 'clubs']
type Suit = SuitTuple[number];  // "hearts" | "diamonds" | "spades" | "clubs"
```
## Union to intersection

>- [Type inference in conditional types by ahejlsberg · Pull Request #21496 · microsoft/TypeScript](https://github.com/Microsoft/TypeScript/pull/21496)
>- [typescript - Transform union type to intersection type - Stack Overflow](https://stackoverflow.com/questions/50374908/transform-union-type-to-intersection-type)

```js
type UnionToIntersection<U> = (U extends any ? (k: U) => void : never) extends ((k: infer I) => void) ? I : never;

type Result = UnionToIntersection<T1 | T2>; // T1 & T2
```

## How to build a type inheritance class and include its static fields

```ts
type Flatten<T> = { [K in keyof T]: T[K] }

class Animal {
  static animalStaticField = 'foo';
}

interface Dog extends Flatten<typeof Animal> {}

// Property 'animalStatic' is missing in type '{ prototype: typeof Animal; }' 
// but required in type 'Flattern<typeof Animal>'.(2741)
const dogA: Dog = {prototype: Animal} 
```
## How to ensure that the generic type is an Class (Object) type and not a primitive type

```ts
export type Prop<T> = { new (...args: any[]): T & object }
```
{primitive type} & object will be never type







