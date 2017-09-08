# chrome 插件 [Octo Mate](https://github.com/camsong/chrome-github-mate) 源码分析

## manifest

### permissions 权限

各种 permission 对应功能范围可以在 [Permission Warnings - Google Chrome](https://developer.chrome.com/apps/permission_warnings#warnings) 查到

```
  "permissions" : [
    "*://github.com/*",
    "tabs",
    "webNavigation"
  ],
```

- **tabs** `chrome.tabs` `chrome.windows`
- **webNavigation** `chrome.webNavigation`
- **\*://github.com/\*** 只有 gitub.com 的页面才有权限访问 chrome api

### web_accessible_resources 资源白名单

```
  "web_accessible_resources": ["page.js"],
```

在 version 2 manifest 中，只有配置在这里的资源才能在 web page 跨域访问

![20170907150477037043180.png](http://odoiwipoe.bkt.clouddn.com/20170907150477037043180.png)

在 web page 的 devtools 中跨域访问 `page.js` 成功，但是访问其他资源就不行了

### content_scripts 页面脚本

```
  "content_scripts": [
    {
      "matches": ["https://github.com/*"],
      "run_at": "document_end",
      "css": ["style.css"],
      "js": ["jquery-2.1.3.min.js", "page.js", "config.js", "script.js", "show-size.js", "tab-size.js", "panel.js", "ignore-spaces.js"]
    }
  ],
```

- ` "run_at": "document_end"` 表示在 Dom ready 和 window.onload 之间
- `js` 会按照数组顺序加载执行

### options_page 配置页面

## content script 代码分析

### page.js

将 document 的 `pjax:end` 事件转成 not bubble, not cancelable 的 `_pajax:end` 时间

「疑问？」他是知道 github 在某些时候会触发 `pajax:end` 事件么

「答」[defunkt/jquery-pjax: pushState + ajax = pjax](https://github.com/defunkt/jquery-pjax) 原来就是 github 创始人的作品，github 目前也是用 pjax 的技术，所以在 github 内浏览器页面，加载后都会触发 pjax:end 事件，这都行！

TODO pr【bug】page.js 会 load 两次

### config.js

获取用户配置，merge 默认配置

### scripts.js

- 「增加 UI」

	+  增加目录页面文本文件图标 hover popt
	+ 增加文件详情页面，download 按钮

- 「增加 UI」的时机
	+ js 加载
	+ _pjax:end 事件（每次局部页面刷新）
	+ popstate 事件（浏览器前进后退，pushState/replaceState不会触发这个事件）

- 「增加的事件」

	+ 目录页面点击文本图标

- 「下载机制」
	+ 借助 A 标签的 download="文件名" 属性，浏览器就会下载 href 链接，如果页面没有 A 标签，那么可以创建一个，触发 click 事件


- 「通信获取配置是否有效」

`chrome.runtime.sendMessage({key: "feature-1-enable"}, function(response) {`

内部发起通信，估计是配置管理器接收到 key 为 xx 的信息，就 reponse 一个 true/false 之类的，
然后这里如果没有启动相关功能，则不去注册相关事件

### show-resize.js

主要用来显示：下载包大小、ghpage按钮、缓存仓库信息

哈哈！有的项目会在 summary-bar(` .overall-summary`) 底部显示黄条 `.overall-summary-bottomless`

![](http://odoiwipoe.bkt.clouddn.com/20170907150478248861795.png)
点击黄条会出现下面
![](http://odoiwipoe.bkt.clouddn.com/20170907150478249645836.png)

这个文件是否启用的标准就是有没有 `.overall-summary-bottomless`，这就尴尬了，

### tab-size.js

配置切换 `tab` 字符表示的空格，原来 css 还有 `tab-size` 属性 [tab-size - CSS | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/tab-size)，不过 IE 不支持

### panel.js

创建右侧收藏列表、outline 列表

### ignore-space.js

在 提交、PR 等 diff 页面可以按钮，切换「是否隐藏差异中的空白字符」，通过 url 带上 query `w=1`  实现（这是 github 支持）

总结一句：content scripts 就是增强 web page 页面上的交互

## bg script

```
//background.html
<img id="icon" src="icon.png">
<canvas id='canvas' width='19' height='19'></canvas>
<script src="config.js"></script>
<script src="background.js"></script>
```

主要处理两件事情：「管理保存配置」「处理 unread badge」

### 管理保存配置

这里引用了 config.js，所以要在一个独立于 web page 的 context 中保存配置，对比一下 web page 和 bg page 的 localstorage

web page:
![](http://odoiwipoe.bkt.clouddn.com/20170908150483797047178.png)

bg page:
![](http://odoiwipoe.bkt.clouddn.com/20170908150483802850597.png)

`gm_config` 保存在 bg page 的 localstorage

### 读取 unread

通过获得 `https://github.com/notifications` 这份 html 页面，解析得到


## options page 配置页

从 `options.js` 看到，它也是写 local storage 的 `gm_config` 来保存，所以跟 bg 是同一个 context











