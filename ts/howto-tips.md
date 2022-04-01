## giving specify the tuple type explicitly and derive the union from it
```js
const ALL_SUITS = ['hearts', 'diamonds', 'spades', 'clubs'] as const;
type SuitTuple = typeof ALL_SUITS; // readonly ['hearts', 'diamonds', 'spades', 'clubs']
type Suit = SuitTuple[number];  // "hearts" | "diamonds" | "spades" | "clubs"
```
## union to intersection

>- [Type inference in conditional types by ahejlsberg · Pull Request #21496 · microsoft/TypeScript](https://github.com/Microsoft/TypeScript/pull/21496)
>- [typescript - Transform union type to intersection type - Stack Overflow](https://stackoverflow.com/questions/50374908/transform-union-type-to-intersection-type)

```js
type UnionToIntersection<U> = (U extends any ? (k: U) => void : never) extends ((k: infer I) => void) ? I : never;

type Result = UnionToIntersection<T1 | T2>; // T1 & T2
```
