## init	820c2cf	Evan You <yyx990803@gmail.com>	2020年4月20日 下午4:00

大致思路：监听本地文件、拦截请求、浏览器热更新

启动一个 server，通过 vue 相关中间件对于 .vue 文件的请求拦截，返回编译改造后的 vue 组件代码

监听本地 .vue 文件，如果有改动，通过 websocket 发送事件实现热更新等

页面需引入一个 js，用于创建 ws，监听 server 发送的一些事件，执行相应更新操作


vue 中间件：

对于一个 .vue 文件
```
<template>
  <div class="name">xx{{ name }}</div>
</template>

<script>
export default {
  data() {
    return { 
      name: 'peter'
    }
  }
}
</script>

```

通过 http://localhost:3000/demo.vue 后，会得到

```
const script = {
  data() {
    return { 
      name: 'peter'
    }
  }
}

export default script
import { render } from "/demo.vue?type=template"
script.render = render
script.__hmrId = "/demo.vue"
```

访问 /demo.vue?type=template 这种带有参数的连接，那就返回带有 render 方法的代码

```
...
export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", _hoisted_1, "xx" + _toDisplayString(_ctx.name), 1 /* TEXT */))
}
```

## feat: auto inject hmr client	4a04d81	Evan You <yyx990803@gmail.com>	2020年4月20日 下午4:45

`parseSFC` 方法默认不走缓存，考虑热更新的场景，如果走缓存就没办法拿到最新的

# feat: module rewrite	33488fe	Evan You <yyx990803@gmail.com>	2020年4月20日 下午5:32

变量名带上 __ 前缀

`moduleRewriter.js` 专门改造 sfc-compile 生成的代码:
`.vue` 返回的 script 不是简单的拼凑，通过 babel/parser 解析，找到 `import` 和 `export default`，对于 import npm 包的，统一改为 '/__modules/xx'

`moduleMiddleware.js` 就是返回 node_module 包的代码

## refactor: use async fs + expose createServer API	bfb4b91	Evan You <yyx990803@gmail.com>	2020年4月21日 上午6:14

读取文件 fs 改成异步

resp 返回文件内容改成 stream

模块查找基于 cwd 路径

对于 __modules/xx 请求 return moduleMiddleware(pathname.replace('/__modules/', ''), res)

## refactor: use TS	91d76bf	Evan You <yyx990803@gmail.com>	2020年4月21日 上午7:13

@npm
- `npm-run-all` 并行/串行 执行 npm script

ts 配置
```
{
  "compilerOptions": {
    "sourceMap": false,
    "target": "esnext",
    "moduleResolution": "node",
    
    "esModuleInterop": true, // 增加一些辅助代码，保证 import default 和 import * 对于 commonjs 代码不会出错
    
    "declaration": true, // 生成 d.ts
    "allowJs": false,
    
    "allowSyntheticDefaultImports": true, // false 则：对于没有 export default 的 xx.js ，那么 import x from xx.js 就会报错
    
    "noUnusedLocals": true,
    "strictNullChecks": true, // null和 undefined值不包含在任何类型里，只允许用它们自己和 any来赋值（有个例外， undefined可以赋值到 void）
    "noImplicitAny": true, // true: 有隐含的 any类型时报错
    "removeComments": false,
    "lib": [
      "esnext",
      "DOM"
    ]
  }
}
```
[typescript - Understanding esModuleInterop in tsconfig file - Stack Overflow](https://stackoverflow.com/questions/56238356/understanding-esmoduleinterop-in-tsconfig-file)

## add tests	e718bd5	Evan You <yyx990803@gmail.com>	2020年4月21日 上午8:57

@npm
- `execa` 是可以调用shell和本地外部程序的javascript封装。会启动子进程执行。支持多操作系统，包括windows。如果父进程退出，则生成的全部子进程都被杀死



`test/test.js`
```
server.kill('SIGTERM', {
  forceKillAfterTimeout: 2000
})
```
SIGTERM 程序结束(terminate)信号，与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出。shell命令kill缺省产生这个信号。SIGTERM is the default signal sent to a process by the kill or killall commands.

所以这里 forceKillAfterTimeout: 2000 就表示，如果 2 秒了， server 进程还没自己正常退出，那就强杀

整个测试的思路就是，准备一个项目
```
xx.vue
main.js
index.html
```
然后拷贝到临时目录，启动 server，用 puppeteer 做 e2e

## remove git add from lint-staged	3e64a74	Evan You <yyx990803@gmail.com>	2020年4月21日 上午8:58

## properly handle cwd	93286b9	Evan You <yyx990803@gmail.com>	2020年4月21日 上午10:04

@npm
- `resolve-from` Resolve the path of a module like require.resolve() but from a given path, 代替 `resolve-cwd`
- `minimist`轻量命令行参数解析器

### `moduleResolver.js`

增强了  module 查找能力

```
modulePath = resolve(cwd, `${id}/package.json`)
```

支持 `.map` soucemap 文件

### `server.js`

createServer 带上 cwd 参数，后续所有跟启动路径相关的地方都传入，这样别的地方就不存在 `process.cwd()`

## refactor types	f4382f1	Evan You <yyx990803@gmail.com>	2020年4月21日 上午11:08


`vueCompiler.js` 把 compile script 和 compile template 封装

## feat: style hot reload	140f2b2	Evan You <yyx990803@gmail.com>	2020年4月21日 下午12:37

@npm
- `hash-sum` blazing fast unique hash generator; 输出一个变量/function，计算 hash

### `watcher.js`

style hotreload 的处理：

descriptor.styles 前后两次比较，
- 如果有 `scoped` 改变，则发送 `reload`事件
- 顺序遍历比较， 如有不同发送 `style-update' 指定索引
- 新的更长，则旧的部分删除，发送 `style-remove` 指定 `{id: `${hash_sum(resourcePath)}-${i + nextStyles.length}`}`

比较两个 block 是不是一样:
- 强等于比较
- keys 长度比较
- 遍历 keys 比较属性

### `vueCompiler.js`

`compileSFCMain()`

对于有 scoped 的 style block，请求 `.vue` 的代码包含
```
import "xxx?type=style&index={i}&${timestap}"
```

`compileSFCTemplate()`

带上参数 id: `data-v-${id}`, 其中 ${id} 就是请求的 `pathname`

`compileSFCStyle()`

请求 type=style，就编译，拼接创建 style 标签的代码，标签 id 为 `vue-style-${id}-${index}`


