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






