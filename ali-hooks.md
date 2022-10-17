# ali-hooks阅读笔记

## ****useMemoizedFn****

useMemoizedFn( fn, [dep1,…])

### 场景：

在使用 useCallback 来保证一个函数 prop 的地址不变，避免无谓重复渲染，当遇到需要 deps 有变化的时候，useCallback 也会返回新的地址，保证函数内引用最新的 deps。

useMemoizedFn 就是保证返回的地址不变，然后函数内也能访问到最新的 state。

### 实现：

1）考虑返回地址不变，那么 hook 返回的需要一个 Ref 值

2）Ref 的 current 只赋值一次，为一个函数，函数内再去调用另一个 Ref<Fcuntion>

3）内 Ref 函数则在组件函数内每次都设置为最新的函数，保证闭包访问最新的 state

总结：一个ref保证返回地址不变，另一个ref可以保证持续访问到最新的函数引用（从而访问最新的闭包内容）

## ****useCreation****

useMemoizedFn( factoryFn, [dep1,…])

### 场景：

等同于语义上的 useMemo，React 文档提到，不能完全依赖 useMemo 语义上的意思来做性能优化，未来有可能会重新计算，例如处理离开屏幕的组件，重新回到屏幕之后做计算。另外比起 useRef 还有一个优势，useRef( factoryFn) 传入一个构造器，其实每次都会执行构造器，只是会把后续传入的值忽略，相当于是 `const a = factoryFn(); useRef(a)` 

### 实现：

1）useRef 一个引用对象，存放三个属性：返回值、是否初始化、依赖数组

2）重新计算时机：首次初始化，或者依赖有变化时
