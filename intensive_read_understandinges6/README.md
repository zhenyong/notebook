# 第十三章 模块

## 为什么要有模块

回忆一下过去我们 js 的文件加载后，代码加载的变量方法，本质上都在同一个全局共享空间，我们通过一些编码习惯，约定一些命名空间，避免了一些代码污染等问题，那么 ES6 的模块，帮助我们解决作用域的问题

## 什么是模块

1. 默认 strict mode，没得改
- 模块内顶级作用域的变量不会添加到全局，只存在于模块内顶级作用域
- 模块内top作用域里的 this，是 undefined
- 不支持 html 注释风格

    ```
    \<script\>
        <!-- 
            老的浏览器可能不支持 js，于是把js 代码用一个 html 注释包起来，
            如果能支持 js 的，那么html注释的闭合部分就被注释掉了，那么 js 自然就执行
            如果不支持 js，那么 html 注释闭合就是把 js 代码注释掉了，自然就不报错

         //-->
    \</script>
    ```

    现在的浏览器还有不支持 js 么，这个古老的东西， es6 就抛弃了

- 想要给外部使用的一定要 export
- 模块可以引用其他模块的binding //TODO 这里binding 用词是什么意思，我以为就是模块中的功能罢了

## 简单 export


    ```
    // 模块代码
    export const name = x
    export function foo () {}
    export class A {}
    function bar (){ }
    export {
        bar
    }

    // 使用外部模块代码
    import { name, foo, bar } from 
    // 此时 name, foo.. 都是 read-only，不能赋值，像 const

    import * as x from 
    // 此时 x 就跟普通定义的对象一样了，可以访问 x.name, x.foo，也可以修改赋值
    ```

直接在带名称 function / class / 变量前面加 export，或者直接 export 一个对象, key 就是名称，

## 模块的怪异

如果 A 模块 import B 的变量，在 A 里面不能直接修改，但是如果 B 模块方法调用引起的 B 的变量发生变化，那么在 A 里面也会跟着同步

## import 到不同模块，同名如何处理

```
import { name as nameForB }

export { username as name}
```

## export 已经 import 的

```
export { sum } from 'x.js'

等同于

import { sum } from "x.js";
export { sum }
```

```
export * from "x.js"
这里会忽略 x.js 里面的 defualt
```

如果需要 export 别的模块里面的 default，就要import 之后再去声明 export

## 浏览器中 module script 的执行时机

默认以 defer 行为加载执行

- 按顺序执行
- 保证所有依赖资源（import），加载完之后再执行


