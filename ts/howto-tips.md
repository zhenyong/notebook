## giving specify the tuple type explicitly and derive the union from it
```ts
const ALL_SUITS = ['hearts', 'diamonds', 'spades', 'clubs'] as const;
type SuitTuple = typeof ALL_SUITS; // readonly ['hearts', 'diamonds', 'spades', 'clubs']
type Suit = SuitTuple[number];  // "hearts" | "diamonds" | "spades" | "clubs"
```
