## 判断一个函数是否通过 new 执行

```
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```

## 运行时的性能监控

[Performance - Web API 接口参考 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance)

```javascript
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
//...
mark('compile')
const { render, staticRenderFns } = compileToFunctions(template, {
//...
mark('compile end')
measure(`vue ${this._name} compile`, 'compile', 'compile end')
//...
}


const perf = inBrowser && window.performance
if (
  perf &&
  perf.mark &&
  perf.measure &&
  perf.clearMarks &&
  perf.clearMeasures
) {
  mark = tag => perf.mark(tag)
  measure = (name, startTag, endTag) => {
    perf.measure(name, startTag, endTag)
    perf.clearMarks(startTag)
    perf.clearMarks(endTag)
  }
}


```
