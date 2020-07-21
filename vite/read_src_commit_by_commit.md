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

