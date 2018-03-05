# read-parcel-src
## 1de7d78 Initial commit
一眼看到一个 `test.js`

```
const Bundle = require('./src/Bundle');

async function run() {
  let bundle = new Bundle('/Users/zhenyong/codes/parcel/test.js');
  let module = await bundle.collectDependencies();
  printDeps(module);
}
...
``
```
给定文件路径，创建一个 `Bundle` 对象，然后收集依赖信息

进去 `src/Bundle.js` 看看，主要流程:

1. 把 main 文件解析成 Module 对象
2. 通过 worker 解析计算 main Module 的依赖
3. 对 main Module 的依赖进行 #1#2

进去 `src/Resolver.js`

根据一个包名/文件路径，返回最终对应的「main」文件路径

```
const builtins = require('node-libs-browser');
...
    builtins[key] = require.resolve('./_empty.js');
```
如果 `node-libs-browser` 给出的映射中，有些内置 node 包没有相应 browser 实现，则统一指定为一个空文件的路径

``

`src/Parser.js`

管理不同文件类型的 parse 方法

`visitors/dependencies.js`

遍历 ast 语法树的时候，提供一些回调

```
 {
	ImportDeclaration(node, module) {  // import from '记录这个路径'

    ExportNamedDeclaration(node, module) { // export { xx } from '记录这个路径'

    ExportAllDeclaration(node, module) { // export * from '记录这个路径'

    CallExpression(node, module) { // require('记录这个路径')
};
```

到此，流程就很清晰了，看下一些工具方法啥的

`utils/*`

其中 `promisify.js` 就是把传统 callback 转成 promise 方式使用，其实 Node 提供 `util.promisify`，不用自己写，另外 `queue.js` 暂时没用到，其实我也经常自己手写 queue，每次重写，没有沉淀封装起来，不应该呀！

### 涨知识
`/src/Bundle.js`

[rvagg/node-worker-farm: Distribute processing tasks to child processes with an über-simple API and baked-in durability & custom concurrency options.](https://github.com/rvagg/node-worker-farm) 把计算任务分配给子进程（worker）

[Cluster模块 -- JavaScript 标准参考教程（alpha）](http://javascript.ruanyifeng.com/nodejs/cluster.html)

`/src/Resolver.js`

使用 [defunctzombie/node-browser-resolve: resolve function which support the browser field in package.json](https://github.com/defunctzombie/node-browser-resolve) 获得一个「包」/「文件路径」对应的「package.json」信息，也就是依赖信息

[webpack/node-libs-browser: The node core libs for in browser usage.](https://github.com/webpack/node-libs-browser)给出一个hash obj：<node 内置包名，对应的文件路径（browser implementation）>

`parsers/`

[babel/packages/babylon at master · babel/babel](https://github.com/babel/babel/tree/master/packages/babylon) Es Next 代码解释器

`/src/Module.js`

[pugjs/babylon-walk: Lightweight Babylon AST traversal tools.](https://github.com/pugjs/babylon-walk) 比 `babel-traverse` 轻量的 ast 遍历器

## b6a73ab Refactor parsers -> assets
删除了 `Module` 类，引入一个 `Asset` 抽象类，构造方法跟原来 `Module` 类实现区别不大，parser 解析之后返回一个 `Asset` 实例，原来解析首先找到某个文件类型的 parser 方法，然后返回执行内容，现在是直接返回相应 `Asset` 子类实例，意味着每个文件对应一个 `Asset` 实例，每种文件对应一种 `Asset` 子类，子类实现相关抽象方法。

`parsers/**` --> `assets/**` 把解析方法上升到资源类型的概念

`worker.js` 屏蔽了读取代码，调用 parser 的参数等细节，直接调用通过 `asset` 实例 collectDependencies 方法

## 12ae01d Run babel transform when needed
`Resolver` 返回更多信息，包括 package.json (pkg) 内容

`Asset` 构造参数也包括 pkg 信息

`worker.js` 中 babel transform 封装成单独方法，便于过滤无须 babel transform 的，例如：json、根目录、node_module 目录

`worker-farm` 原来直接使用，现在封装了一层，扩展事件 emit 机制，目前还没看到使用场景

## 54b5299 Skip loading dependencies for assets that don’t have import/require
`JSAsset` 重写父类方法 `getDependencies`，如果代码明确没有依赖，则不执行

## 1e17e7a JSON always has no dependencies. No need to parse it.
`JSONAsset` 重写父类方法，不收集依赖

## 2a81299 Fix package.json access from worker
参数补漏

## 302183a Cache fs.exists calls

## 003267a Insert module globals
新增代码解析遍历钩子 `visitors/globals.js`，
主要功能是找到一些全局变量，下面 `VARS` 的 key 表示一些「全局变量」的样子，
主要代码中有相关的变量引用，则执行相关逻辑

```
const VARS = {
  process
  global
  __dirname
  __filename
  Buffer
};
```

### 钩子 Declaration

```
  Declaration(node, module, ancestors) {
    // 如果 If there is a global declaration of one of the variables, remove our declaration
    let identifiers = types.getBindingIdentifiers(node);
    for (let id in identifiers) {
      if (VARS.hasOwnProperty(id) && !inScope(ancestors)) {
        // Don't delete entirely, so we don't add it again when the declaration is referenced
        module.globals.set(id, '');
      }
    }
  }
```

当遇到 `const` 等声明符号，时，进入 `Declaration`，
`types.getBindingIdentifiers` 就是拿到相关变量定义的语法节点，
例如

```
const a = 1, b = 2;
```

则 `identifiers` 为 {a: <ast info>, b: <ast info> }

其中 `VARS.hasOwnProperty(id) && !inScope(ancestors)` 的判断意义：

`VARS.hasOwnProperty(id)` : 如果变量标识属于「全局变量」(VARS 的 key)，

`!inScope(ancestors)` 并且不曾在上级作用域链中声明，具体逻辑

```
function inScope(ancestors) {
  for (let i = ancestors.length - 2; i >= 0; i--) {
    if (types.isScope(ancestors[i]) && !types.isProgram(ancestors[i])) {
      return true;
    }
  }
  return false;
}
```

`ancestors` 相当于一个语法树，尾部元素在上面例子表示 `const` 本身，那么就不用考虑这层，直接从 `.length - 2` 往上遍历，如果在某一层作用域中，并且不是在顶层作用域（声明全局变量），那么返回 `true`

```
        module.globals.set(id, '');
```

标记：不要完全删除这个映射，标识为空，下次就不添加声明

简单来说，如果模块代码中有

```
function xxx() {
  const global = 1; // 声明了「全局变量名称」，但是在该作用域链中找到，则忽略这个「全局变量」
  console.log(global)
}

```

【疑惑】：如果两个方法，一个声明了 global，另一方法没有，怎么破？

### 钩子 Identifier

```
Identifier(node, module, ancestors) {
   let parent = ancestors[ancestors.length - 2];
   if (VARS.hasOwnProperty(node.name) && !module.globals.has(node.name) && types.isReferenced(node, parent)) {
     module.globals.set(node.name, VARS[node.name](module));
   }
 },
```

简单来说，如果某个标识符是「全局变量名」，并且被「使用」，那么就搞事情，也就是提供一个 hack 声明全局变量的方式，因为这类「全局变量」在 Node 环境可以直接访问，hack 一下，避免在别的环境代码报错，从 `worker.js` 中的使用方式就看出来：

```
  asset.contents = Array.from(asset.globals.values()).join('\n') + '\n' + asset.contents;
```

在文件内容前面添加 「hack 声明全局变量」的代码

【疑惑】这些变量会再浏览器环境运行吗？如果不会，为啥源码会带有这些「全局变量」呢？

## 7346f37 Add JSPackager
开始出现「装包器」的概念，`JSPackager` 继承 `Readable`

如果源码 `code.js`

```
function xxx() {
	const a = global + 1;
}
```

打包之后:

```
(function e(modMap, loadedMap, modIdList) {
	// ... 模块加载的处理
})(
 {
 // 模块全局计数 id
   1: [
     function(require, module, exports) {
       var global =
         typeof global !== 'undefined'
           ? global
           : typeof self !== 'undefined'
             ? self
             : typeof window !== 'undefined' ? window : {};
       function xxx() {
         const a = global + 1;
       }
     },
     {
     	// 这里会存放该 `1` 模块依赖的子模块，例如:
     	'./src/_empty.js': 2
     	// 其中 2 就是全局的 模块 id
     },
   ],
 },
 {}, // 在上面模块加载处理时，表示已经加载过的模块map
 [1], // 罗列出入口要加载的模块id（猜的Ï）
);

```
最终是返回一个 load 方法，目前来看没有太特别，跟 webpack 最开始的数据结构差不多，目前只考虑一个入口文件的情况，
这里有个小技巧:

先把模块相关的资源在全局缓存一遍，形成一个「全局性」 map

```
for (let mod of asset.modules.values()) {
 this.addAsset(mod); // this.includedAssets.add(mod);
}
```

后面在遍历模块本身，然后从全局 map 拿到模块id，保证相同资源都是同个模块 id
```
for (let [dep, mod] of asset.modules) {
  deps[dep] = this.includedAssets.get(mod);
}
```

### 涨知识
`console.profile(name); ... console.profileEnd(name)` 浏览器控制台会给出一些性能分析，例如 CPU 使用相关的信息

## 4d26580 module -> asset
一个文件对应一个 asset 的概念

## a964a94 Some refactorings
`JSAsset.js` 重构了，移除了 super.xx 这类代码，把通用逻辑放到父类 A 方法，父类 A 方法在调用 B 方法，子类重写 B，还是很普遍的做法

`worker.js` 把 babel 一些调用细节封装到 JSAsset，通过父类统一方法 asset.transform 来执行资源转换

到这一步已经有了一些转码、生成的感觉了

## 76fc3f5 Support multi-bundle output

> e.g. a bundle for CSS and one for JS. Eventually, with dynamic import, this will support automatic bundle splitting.
如果是你，这时候怎么处理多 bundle 呢？

`Bundle.js` 的一些动作行为放在新的 `Bundler.js`

梳理一下概念

* Asset: 都应一个文件，负责文件本身解析获取依赖信息，还有该类型代码生成方法
* Bundler:
创建相应 bunlde，入口文件基本上就是 rootBundle，递归依赖关联 Asset，遇到当前 bundle 不合适的类型，就创建子 bundle，然后 bundle.pakcage() 开始打包
* Packager: rootBundle 关联的 asset 依赖生成代码，然后递归子 bundle ...

bundle 偏向输出「文件束」的概念，一个入口表示一个根「文件束」，因为入口文件的依赖可能有不同类型的资源要处理，所以对于不同类型的所有子依赖，都统一关联到子 bundle。

## 96bf882 Allow assets to generate multiple representations for different bundles
同一个资源，在不同 bundle 中有不一样的表示？
【疑惑】意思是有的就是正常输出，有的就是占位输出？还是说，像 .vue 文件输出三种类型的内容

`Asset.js`

* 跟 `Bundle` 是多对多
* 把 `asset.ast` 不在外部设置，用 `asset.generated` 表示输出的不同类型的内容

`Bundler.js`

Asset 如果不属于当前 Bundle 的类型，但是对应生成的内容可能是几种类型，如果其中一种属于当前 bundle，也要关联

## ac81ae5 Give assets an id and fix deduping
问题：JSPackager 防止重复输出依赖 asset 的逻辑有问题

1. 缓存map 的值改成全局性的 asset id
2. 有的 asset 可能没有输出 generated.js，就没进去缓存 map，此时应该直接取 asset.id

【疑惑】直接全都取 asset.id 不就好了

## 938c217 Stylus support
`babel.js` 里面找某个包的 .babelrc 方法封装成工具方法

## d5c7645 Implement watcher support

> Only recompiles assets that changed.
`Asset.js` 把依赖解析、代码转换、生成，都封装在一个方法里
`Bundle.js` 打包输出时，可以选择是否要包含孩子，方便后面某个 Asset 修改后，package 的时候不要处理孩子
`Bundler.js` worker 处理进程，在 watch 模式下不关闭

## 9052759 Implement filesystem cache
`Asset.js`  worker 执行解析的逻辑放到外层 `Bundler` 执行，便于缓存

asset 解析出来的结果通过文件缓存，策略就是文件名作 key，根据修改时间判断是否返回缓存

## c248c7f First attempt at code splitting using dynamic import

捋了一遍

[百度脑图－便捷的思维工具](http://naotu.baidu.com/file/bc606e59159d0c03285256962d568bd3?token=11d0252f90ba44d7)

总结几个概念

* Bundle 就是一个「面向输出」的入口文件
* Asset 就是某个代码文件、包（npm包规则）
	* 各种子类实现了对应的解析、依赖收集、代码生成
* Packager 管理 AST 之后的代码组织规则
* Bundler 就是构建工具的执行入口类

## 1bbb98d Don’t package an empty bundle

## 1dc5ce4 Generate asset and bundle hashes

- assset 的 hash 就是解析后各类型的输出 md5
- bundle 的 hash 就是各 asset 的 hash 的 md5

## 18beada Refactor bundle tree creation to support code splitting

- 重构创建 bundle 和 asset 树
- 移动资源到公共 bundle 而非 根 bundle

## e0b4d61 Support dynamic imports

### JSPackager

遇到动态加载的模块，增加依赖，并且把代码中的 `require` 调用替换成 `_bundle_loader`

```
if (isDynamicImport) {
  asset.addDependency('_bundle_loader');
  asset.addDependency(args[0].value, {dynamic: true});

  node.callee = requireTemplate().expression;
  node.arguments[0] = argTemplate({MODULE: args[0]}).expression;
  asset.isAstDirty = true;
}
```





