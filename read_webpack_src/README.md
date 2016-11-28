webpack 源码从头到尾
===

# 2e14600 

> Initial commit

## webpack 加载器解析

源码长这样:

```javascript
var a = require("a");
var b = require("b");
require.ensure(["c"], function(require) {
    require("b").xyz();
    var d = require("d");
});
```

经过 webpack 构建之后：

```
|- output.js
|- 1.output.js
```

每个输出文件叫做一个 chunk，chunk 内的各个模块叫做 module

### 入口 chunk ( output.js )

```
(function(modules) {
    var parentJsonpFunction = window["webpackJsonp"];
    
    // chunk 的异步加载后执行入口，类似 jsonp 的 callback 方法
    window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules) {...};
    var installedModules = {};
    var installedChunks = {
        0: 0
    };
    // 加载 module 的方法
    function __webpack_require__(moduleId) {}
    
    // 加载 chunk 的方法
    __webpack_require__.e = function requireEnsure(chunkId, callback) {};
    
    // 缓存每个 module 对方的方法
    // 初始化为入口 chunk 中定义的 modules
    __webpack_require__.m = modules;
    
    // 已经安装的 modules 信息
    __webpack_require__.c = installedModules;
    
    // 资源访问的基路径
    __webpack_require__.p = "";
    return __webpack_require__(0);
})
([
    /* 0 */ 模块 0 对应 output 本身这个模块
    function(module, exports, __webpack_require__) {
        var a = __webpack_require__(1);
        var b = __webpack_require__(2);
        
        // 这里加载另一个 chunk 1
        __webpack_require__.e /* nsure */ (1, function(require) {
            __webpack_require__(2).xyz();
            var d = __webpack_require__(4);
        });
    },
    /* 1 */
    function(module, exports) { // module a },
    /* 2 */
    function(module, exports) { // module b }
]);
```
### \_\_webpack\_require\_\_

加载模块的方法

```javascript
function __webpack_require__(moduleId) {
    
    if (installedModules[moduleId])
        return installedModules[moduleId].exports;

    // 创建一个新模块，然后放到缓存对象中
    // module 简单理解成 node 的 module 呗
    var module = installedModules[moduleId] = {
        exports: {},
        id: moduleId,
        loaded: false
    };

    // 执行模块对应方法，进行安装
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

    module.loaded = true;

    return module.exports;
}
```

### requireEnsure

```javascript
function requireEnsure(chunkId, callback) {
   	// "0" 表示 "已经加载安装完"，其他情况可能表示加载中
    if (installedChunks[chunkId] === 0)
        return callback.call(null, __webpack_require__);

    // 如果是 0 以外的值，那就是 "当前正在加载的"
    if (installedChunks[chunkId] !== undefined) {
    	// 如果之前处于『正在加载』，那么把回调塞入『加载后』的执行队列
        installedChunks[chunkId].push(callback);
    } else {
        // 表明正在加载
        installedChunks[chunkId] = [callback];

		// 创建 script 标签加载资源
        var head = document.getElementsByTagName('head')[0];
        var script = document.createElement('script');
        script.type = 'text/javascript';
        script.charset = 'utf-8';
        script.async = true;
        script.src = __webpack_require__.p + "" + chunkId + ".output.js";
        head.appendChild(script);
    }
};
```

入口文件本身也算一个 chunk (0),

## webpack 命令

- `single`： false 表示不分割分件（code spliting）
- `min`： false 表示不压缩代码（uglifyjs）
- `filenames`：false 表示不把文件名输出到最终文件
- `options`：指定一个 json 配置文件表示各参数
- `script-src-prefix`：资源路径前缀
- `libary`：指定一个变量名，最终会 export 为这个全局变量

## webpack 源码

```
|
|- webpack.js
|- resolve.js
|- parse.js
```

### webpack.js
		
### resolve.js

解析出文件 或者 node 模块的文件绝对路径

### parse.js

找出文件中的 require 和 ensure 依赖的模块

```javascript
var a = require("a");
var b = require("b");
require.ensure(["c"], function(require) {
    require("b").xyz();
    var d = require("d");
});
```
===>	

```json
{
	"requires": [{
        "name": "a",
        "nameRange": [16, 18],
        "line": 1,
        "column": 8
    }, {
        "name": "b",
        "nameRange": [38, 40],
        "line": 2,
        "column": 8
    }],
    "asyncs": [{
        "requires": [{
            "name": "c"
        }, {
            "name": "b",
            "nameRange": [98, 100],
            "line": 4,
            "column": 11
        }, {
            "name": "d",
            "nameRange": [130, 132],
            "line": 5,
            "column": 12
        }],
        "namesRange": [59, 63],
        "line": 3,
        "column": 0
    }]
}
```
其中 `asyncs` 数组的一个元素对应一个 chunk，这里只有一个元素，意思就是会从该模块文件分出一个 chunk

### buildDeps.js

构建依赖树信息

**模块**：入口文件叫（入口）模块，其他 `require('xx')` 为 `xx` 模块

构建依树的步骤：

```
buildDeps(context, mainModule, options, callback) {
	...
	var depTree = {
		modules: {}, // <filename, module>
		modulesById: {},
		chunks: {}, // <chunkId, {	modules:{moduleIdA:'include', moduleIdB: 'in-parent'...}	} >
		nextModuleId: 0,
		nextChunkId: 0,
		chunkModules: {} // used by checkObsolete
	}
	var mainModuleId;
	# 1
	addModule(depTree, context, mainModule, options, function(err, id) {
		...
		mainModuleId = id;
		buildTree();
	});
	function buildTree() {
		# 2
		addChunk(depTree, depTree.modulesById[mainModuleId], options);
		for(var chunkId in depTree.chunks) {
			# 3
			removeParentsModules(depTree, depTree.chunks[chunkId]);
			# 4
			removeChunkIfEmpty(depTree, depTree.chunks[chunkId]);
			# 5
			checkObsolete(depTree, depTree.chunks[chunkId]);
		}
		callback(null, depTree);
	}
}
```

1. 入口文件（模块）`example.js` 解析开始，深度优先地使用 `parse.js` 对每个模块解析出 `requires` 和 `asyncs`
2. 解析完模块后，通过 `addChunk` 方法，新增入口模块对应的 chunk，在这个过程中，对于模块对象中的 `asyncs` 数组的每个元素，分别增加一个 chunk，每个 chunk 依赖的模块对应相应信息中的 `requires` 包含的模块，并且在 chunk 信息中，对于包含的模块标识一个 *'include'*

	```
	"1": {
	            "id": 1,
	            "modules": {
	                "2": "include",
	                "3": "include",
	                "4": "include"
	            },
	            "context": {
	                "requires": [{
	                    "name": "c",
	                    "id": 3
	                }, {
	                    "name": "b",
	                    "nameRange": [120, 122],
	                    "line": 5,
	                    "column": 11,
	                    "id": 2
	                }, {
	                    "name": "d",
	                    "nameRange": [152, 154],
	                    "line": 6,
	                    "column": 12,
	                    "id": 4
	                }],
	                "namesRange": [81, 85],
	                "line": 4,
	                "column": 0,
	                "chunkId": 1,
	                "chunks": [1]
	            },
	            "parents": [0]
	        }
	```

3. 根据 chunk 的分割父子关系，如果子 chunk 包含的模块在父 chunk 中出现，则 'include' -> 'in-parent'
4. 如果模块为空（找不到文件），则移除
5. 如果多个 chunk 包含的模块一样，则会将其余的 chunk 的 chunkId 等信息设置为其中一个的 chunkId

总结，依赖树包含下列信息:

1. 每个模块的依赖元信息
2. 每个 chunk 依赖了哪些模块，哪些是自己包含，哪些已经在 父 chunk 中
3. chunk 之间的 "父子" 关系
4. chunk 的上下文信息，对应 入口模块 / 分割 ，前者为入口模块 parse 后的信息，后者为 `asyncs` 中的一个元素

### 


## 涨姿势

**npm**

- [substack/node-optimist: Light-weight option parsing for node.js](https://github.com/substack/node-optimist) 命令行参数解析
- [jquery/esprima: ECMAScript parsing infrastructure for multipurpose analysis](https://github.com/jquery/esprima) JS 语法解析

**refer**

- [SpiderMonkey Parser API](https://developer.mozilla.org/en/SpiderMonkey/Parser_API) Esprima 的 API 兼容 SpiderMonkey 的 API


