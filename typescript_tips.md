## 参数是一个类

```ts
// 1
export function foo<T>(
    clazz: new (...args: any[]) => T
): void

// 2
export function foo<T>(
    object: T
): void

class Person {}
const obj = {}

foo(Person, {}) // #1
foo(obj, {}) // #2
```
