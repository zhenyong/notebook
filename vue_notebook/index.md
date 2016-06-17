## [83fac01]

> initial setup

作者使用了 grunt+component

## [871ed91]

> rename

作者在 /explorations 下进行一些数据绑定的实验，我作了分析，在[这里](data-element-binding.md)

## [a5e27b1]

> naive implementation

作者在实验代码加入 rivets.js，这是一个轻量的数据绑定、指令...库，估计是学习别人好的方面；

作者在 /src 下实现简单的指令和过滤器，如果要运行作者的代码需要自己编译，安装 grunt 和 component，可能会遇到问题，参考[这里](build_challenge.md)

[这里](template_analysis_a5e27b1.md)解剖了最初的模板解析、数据绑定的相关代码

## [3eb7f6f]

> filter value should not be written


- 添加 Seed.prototype.destroy 方法

	调用指令 unbind（如果有），然后移除根节点
> sd-on-xx 指令的 unbind 则是移除事件处理器

- 准备进行 repeat 指令实验
- Seed.prototype.dump

		抽取数据 binding = {msg: {
			..,
			value: 'hello'
		}} 
		返回
		{msg: 'hello'}
	
## [ec39439]

> dump, destroy, fix filters

增加过滤器，重构

## [cf1732b]

> refactor

重构

- 指令信息对象抽象成指令类
- 对于 Seed 来说，只存储变量、指令、值的映射关系`bindings`，绑定、更新等具体逻辑封装在指令内部
- 过滤器的调用也封装在 指令 内部
- directives.js 声明指令对象拥有什么方法

其中声明 sd-on-xx 指令

		on : {
			update: fun,
			unbind: fun
		}
则变成指令实例的方法

	dir._update(...)
	dir.unbind(...)

## [154861f]

> augmentArray seems to work

开始实现 sd-each 指令，看代码雏形，应该是通过更改具体数组的 push 方法，拦截处理

## [79760c0]

> WIP

- 添加配置文件 config.js, 用来配置前缀值 `sd` 等
- 添加 seed.js，把一个根元素组件抽象成 Seed 类
- 添加 watchArray.js，通过覆盖绑定数组的方法，从而实现拦截
- main.js 暴露公共方法，如创建指令和过滤器，启动用户定义的各个 Seed 实例化

## [5ce3b82]

> refactor

- 视图组件 Seed 类的概念抽象得很彻底了，构造参数就是根元素（模板） 和 数据
- Seed提供静态方法，用于创建组件、指令、过滤器
	
	Seed.extend 返回组件类，通过原型链拷贝构造继承自父类 Seed

get技能：

经典的拷贝构造原型链继承，会出现下面这段
	
	var F = function () {};
	F.prototype = Base.prototype
	Sub.prototype = new F();
	
以上实现在现代浏览器中，一句搞掂

	Sub.prototype = Object.create(Base.prototype);

## [952ab43]

> kinda working now.

重构让代码跑起来

捋一下代码：

### Seed (el, data)

> 组件类：解析指令，设置指令的功能逻辑，初始化数据

- `el` Dom : 根元素节点
- `_bindings` {} : 绑定变量key、指令实例、元素、值...的关联映射
- `scope` {} : 存放数据、方法，与外部通信
- `_compileNode` () : 对每个指令构造指令实例，更新 `_bindings` 对象


### filters.js

> 存放注册的过滤器 


`{}<filterName, filterFunc>`


### directives.js

> 存放注册的指令名称及所需的执行方法

`{}<dirName, definition = dirFunc/dirObject>`

当 dirObject 是，该对象的方法会赋予指令实例

### Directive (def, attr, arg, key)

> 指令类，一个实例对应一个属性

- `attr` {} : {name:}
- `arg` : sd-on-click 中的 'click'
- `key` : sd-on-click
- `_update` () : 对应 definition[#update方法]
- `update` () : 绑定变量改变时，执行方法 = _update + applyFilters
- `applyFilters` () : 更新dom文本前应用过滤器

### watchArray

> 某些指令的绑定值为数组，用于拦截具体数组的方法，管理监测变化

### config

> 用于配置属性前缀，默认 `sd`

## [3149839]

> sd-each-* works

### 重构:

- 深度遍历根元素 compile 替代查找 compile，估计是考虑性能原因，因为选择器本质上也是一种遍历查找
- 过滤器解析使用正则

### 增加 sd-each-*：

sd-each-* 元素下，根据数组的元素个数构建多个子组件，之后会把数组中对应的元素替换为组件的 scope

只是实现了初始化构建，还没实现数组元素增删等监听处理

### *get技能*

	<div sd-on-click="changeMessage | delegate .button">

代理子元素事件是通过过滤器实现

	delegate: function (handler, args) {
		// handler 就是绑定变量的值，即 changeMessage 方法
        var selector = args[0]// '.button'
        return function (e) {
            if (e.target.webkitMatchesSelector(selector)) {
                handler.apply(this, arguments)
            }
        }
    }

sd-on-* 指令的 update 相当于注册事件处理器，而这个处理器有可能经过 delegate 过滤器包装，酷~

# [23a2bde]

>  big refactor.. again

大重构

### Seed

暴露 Seed.controller, 返回的类 等同于 Seed.extend 的返回，

### controller

目前代码看，就是 Seed.extend，controller 的概念还要看后面代码

### Directive => Binding

- 指令类的概念变为绑定类
- key 的解析支持更多格式 （sd-class-red="error" => sd-class="red:error"）
- arg 的解析变更： sd-xx-arg="key" => sd-xx="arg:key"
- 增加 key 表达式校验，忽略不靠谱的指令
- dirInstance对应arr => bindingInstance对应directiveName
	
	从代码上看，支持这种解析
	
		sd-dir="exp1, exp2"
		// attr 为 {name:'sd-dir', expressions:[exp1, exp2]}
		// Seed 内部构造两个 Binding 实例
		// Binding ('dir', 'exp1')
		// Binding ('dir', 'exp2')
	
	于是 `sd-on` 就可以支持多个事件了

### sd-each

构建子组件时，如果有指定 sd-controller 则用它，否则构造一个 Seed 对象

*get技能*

	 if (delegateCheck(e.target, e.currentTarget, selector))
	 	handler.apply(this, arguments)
	
	function delegateCheck (current, top, selector) {
	    if (current.webkitMatchesSelector(selector)) {#3
	        return true 
	    } else if (current === top) {#1
	        return false
	    } else { #2 - 递归到外层事件元素 退出
	    			#4 - 递归到代理目标 退出
	        return delegateCheck(current.parentNode, top, selector)
	    }
	}
	
e.currentTarget 表示事件注册所在元素，e.target 表示真正触发事件的元素，可能是被代理的子元素。

delegateCheck 的三个条件判断表示：

	el -- 事件注册在这里 #1
		el>div				#2
			el>div>a -- 代理这个元素	#3
				el>div>a>button	#4

# [62b75d4]

> kinda work now except for scope nesting

## Seed

### scope chain

		<li 
	        sd-each="todo:todos"
	        sd-on="click:changeMessage, click:todo.toggle"
	        sd-text="todo.title"
	    ></li>

	每个 li 都有自己 scope 对于形如 `todo.xxx`, 如 `todo.title` 这个key，在绑定时，使用自身 scope， 对于 `changeMessage` 这种不带 `todo.` 的则用父 seed（的scope）

### _createBinding 签名重构

	seed._createBinding(key, targetScope)
	 => 	
 	targetSeed._createBind(key)

这就对了，把绑定的概念放在 seed 上，而不是直接关心 scope，况且 scope 就是 seed 的东西

### controller 概念退化

之前代码表现出 controller 是 Seed 的子类，现在不是了，controller 更像一堆纯行为方法，组件还是使用 Seed 去构造，只是在构造过程拷贝对应 controller 的方法，于是 Seed 对象具有了指定 ctrl 的行为方法

### scope 初始化

原先 scope 只拷贝绑定相关的用户数据，现在所有传进来的 data 都拷贝

我想这样也是靠谱的，scope 的方法可能用到绑定无关的用户数据

### _compileNode

之前会深度遍历compile 各个节点，现在从 Seed 实例对应的根节点深度遍历时，如果遇到子节点带有 ctrl 声明则会跳过。

我理解：compile 作一些绑定，是跟作用域 (scope)息 息相关，跳过嵌套 ctrl, 意味着作用域 从概念上看是通过 ctrl 层级来区分所要绑定的 scope （是哪个 Seed 对象）。

疑问：目前例子里面，嵌套 ctrl 刚好作用在 sd-each，而 sd-each 遍历集合创建子 Seed 对象的时候会作绑定。那么，假如 嵌套 ctrl 不是作用在 sd-each 上则没机会作绑定，除非手动 new Seed，看后面代码吧

## sd-on

最初一个 dirInstance 可能管理多个事件处理器，改成 bindingInstance 之后，一个 binding 对象就只对应一个处理器

## sd-each

原来通过 sd-block 标志防止重复 compile, 现在是编译时移除指令属性，反正扫描不到就不会再编译了

构建子 Seed 对象时，传递 parent Seed 而不是 parent Scope

---

*get技能*

	// clone attributes because the list can change
	var attrs = map.call(node.attributes, function (attr) {
	    return {
	        name: attr.name,
	        expressions: attr.value.split(',')
	    }
	})

之前没特别意识到，这段代码很重要，node.attributes 或者 length，每次访问都是实时的，所以通常要先拷贝一份

# [15ffaa4]

> almost there! scope nesting

小重构，变量命名更语义， 例如 seed => scopeOwner，不用看后面代码就知道这是 seed 是父 seed, 跟 scope 传递有关系


#[08e7992]

> finally workssssss

compile 深度遍历，先解析子元素的指令（除了 sd-each）

捋一下碰到 sd-each 时候的解析逻辑

	<li class="todo"
                    sd-controller="Todo"
                    sd-each="todo:todos"
                    sd-text="todo.title"


        if (eachExp) { // each block
            ...
            解析 sd-each 绑定
            ...
            //下面是提前初始化
            self.scope[binding.key] = self._dataCopy[binding.key]
            delete self._dataCopy[binding.key]
            }

        } else if (!ctrlExp || root) { // normal node (non-controller)

            //递归compile子元素

				// 解析绑定其他 sd-*
        }
 
优先判断是否 sd-each, 处理完 sd-each 之后去除这个属属性，然后初始化，此时 sd-each 的 update 执行创建子 Seed, 子 Seed 执行自己的 compile 根节点逻辑，这时候，这个节点的其他 sd-* 进行解析绑定。

疑惑：这个提前初始化是否必要？跟原来：等待所有 compile 完成后再初始化相比，提前初始化能让子 Seed 更早创建，除此之外，还没感受到注释说的
	
	// need to set each block now so it can inherit
    // parent scope. i.e. the childSeeds must have been
    // initiated when parent scope setters are invoked

通过最近两次提交，还没感受到 scope chain 的最终机制，涉及到嵌套 ctrl 的地方还没落实

# [83665f9]

> milestone reached, update todo

记录todo，作者准备解决之前说到的内嵌 ctrl 如何 scope chain 的问题

# [1d7a94e]

> emitter

make seedInstance emitter

# [a6b7257]

> todo demo kinda works

## binding.js => directive-parser.js

换回 Directive 类，囧~

## directives

- sd-on 的事件处理器包装一下，包装参数
- 增加 sd-checked 绑定变量跟input元素checked值绑定

## Seed

- 是否遍历孩子，取决于 !eachExp && !ctrlExp

---
*get技能*

多处重构体现了职责单一原则的开发思想，传递参数时不要多作包装，如果你的包装是为了对方，可能是你知道了对方的业务，那是越界了

# [fcd9544]

> awesome

更新 sd-class

- sd-class="filter" ， 如果scope.filter 为真则 class="filter"
- sd-class="confidition:filter"，如果scope.confidition 为真则 class="filter"

# [f1ed54b]

> nested controllers

## main

- 通过 api={} 对象来暴露公共方法，比起 Seed 的静态方法更简洁
- bootstrap：找到最外层 sd-controller 的元素，然后 new Seed
- 嵌套的 ctrl 会 new Seed，终于解决之前的疑惑
- 增加嵌套 ctrl 中访问父 Seed 的成员
- sd-text
	
		<div sd-data="test"
		Seed.data('test', { todos: todos })
		
	new Seed 的时候如果没有传 {data:xx} 则会去数据中心根据 sd-data 指定的 *test* 键值去获取数据


有一些新的认识，帮助理解：

- 指令: 从英文 directive 看，联想机器指令，一个指令就是做一件事情,对应带代码里的 update
- 绑定：指令做事情，需要材料，一次绑定的过程就是要确定什么数据变化会引起什么指令执行，并且执行过程中的数据主体是谁

# [14d0cef]

> solved the setter init invoke

### 初始化设置

	this._binding = {
		//vvv 某个key的绑定对象
		keyA: {
			value: this.scope[keyA],
			instances: [dirA, dirB]
		}
		//^^^ 
	}

在绑定 key 和 某个 dir 时，会设置绑定对象的 value, 完成绑定关系之后，会使用这个值执行指令的 dir

移除了原来在 Seed 构造最后会统一初始化 scope 的逻辑

# [6d81bff]

> better unbind/destroy

- sd-each: 抽取 unbind 销毁逻辑
- sd-on: 事件包装参数中传谁的 seed
	
	包装处理器时： `self.seed` => `e.currentTarget.seed`，概念上，传递事件绑定元素的 seed
	
	如果代理事件，则在过滤器指定代理元素的 seed `e.seed = target.seed`

# [8f79a10]

> fix sd-on
 
 作者怀疑事件代理的真实性能，移除 delegate 过滤器。我个人也怀疑，除了像表格那类，代理每行dom这样的场景，其他的情况性能可以提高多少呢？别有限考虑性能也是对的，为了它除了很多细节问题，消耗时间，不过 delegate 过滤器的设计确实优雅巧妙。
 
# [3d33458]

> computed property progress

- _compileNode 中条件优化，合并一些 if/else，考虑三类节点

	+ sd-each
	+ 非根部的ctrl
	+ 其他节点（解析指令 & 递归）

- 从表达式解析逻辑看出一点解析深层属性的逻辑

# [5227248]

> event delegation in sd-each

呵呵，刚说了就做了

不过这里事件处理器还有很大重构空间

# [ca62c95]

> break directives into individual files

重构指令代码结构，部分指令单独一个文件，可以扯一下开闭原则，^_^

# [88513c0]

> arrayWatcher

`sd-each` 拦截数组修改方法，实现元素增删响应视图增删，不过 splice 的实现里面，定位 index 的计算公式有疑惑

# [343ea29]

> allow overwritting mutationHandlers

# [c4f31a5]

> better dep parsing

重构，把key、filter 解析单独成方法

# [5acc8a2]

> computed properties!!!

## 依赖绑定

捋一下关系

	<div sd-text="remaining < completed"
	<div sd-text="completed"

其中 remaining 依赖 completed (我们称呼 remaining is dependent )

	_bindings = {
		remained: {
			value: xx,
	       instances: [sdTextDir4Remained]
		},
		completed: {
			value: xx,
			instances: [sdTextDir4Completed],
			dependents: [sdTextDir4Remained],
			refreshDependents: () => {
				each d in dependents
					d.refresh
					// 会调用 sdTextDir4Remained.refresh()
			}
		}
	}
	
	// 看一下，如果 completed 更新了，则...
	set: function (value) {
	    ....
	    if (binding.refreshDependents) {
	        binding.refreshDependents()
	    }
	}

简单说，就是 a 依赖 b, 则 b 更新时会调用 a 更新

疑惑：update 可能导致两次 _update

变量名也是变来变去，重构的过程也是不断打磨，以前经常对自己变量名的频繁变更很烦，看来大可不必。

# [dc04a1a]

> todos

# [7d12612]

> complete todo example

- 增加 sd-value, 目测用来绑定 input 标签
- 增加filter: `key`-过滤键盘键 `currency`
- 初始值为 null/undefined 一样会调用 update

到这次提交，todo demo 完成得差不多，总结捋一下整个开发过程：

- 测试双向绑定核心机制
- 基本的指令、过滤器机制
- 重构、代码文件
- scope chain
- 处理 sd-each
- array watch 机制
- 计算依赖变量

不断更新 TODO.md 的习惯，每一步都清楚目标，代码不多思考编码的时间不少，加油。

# [19b3926]

> jshint;

# [f19e6c3]

> todo

# [c1c0b3a]

> thoughts

# [9a4e5d0]

> use emitters

- sd-each中 array watch 的逻辑移到 seed 中，统一对数组值作拦截
- 支持 value 动态，scope.key = {get: func}


# [f6d6bba]

> sourceURLs for dev, reverse value

13年8月，貌似那会 gulp 有点火的苗头而已，呵呵

# [67ff344]

> add simple example & manual refresh of computed properties

疑惑: refresh 和 update 除了后者多了保护返回，其他几乎一样，后续留意

# [7a0172d]

> auto parse dependency for computed properties!!!!!

## 自动解析依赖

	scope.completed = {get: function () {
        return scope.total - scope.remaining
    }}
    // completed 依赖了 total 和 remaining
   	
自动设置 total 和 remaining 的 dependents 为 completed

规定有依赖的变量都是用 {get: fun}  定义，然后在初始化的时候，就能收集到这部分有依赖的绑定。

	get: function () {
            // debugger;
            if (parsingDeps) {
                depsObserver.emit('get', binding)
            }

每次访问绑定变量都会触发一个事件
	
	Seed.prototype._parseDeps = function (binding) {
	    depsObserver.on('get', function (dep) {
	        if (!dep.dependents) {
	            dep.dependents = []
	        }
	        dep.dependents.push.apply(dep.dependents, binding.instances)
	    })
	    // 调用这句就会进入 get 方法里，get 方法体里面每次使用
	    // 别的绑定对象就会触发 'get' 事件，在上面事件处理器
	    // 去设置 dependents
	    binding.value()
	    depsObserver.off('get')
	}

# [dada181]

> fix _dump()

# [faf0557]

>no longer need $refresh

# [c0a65dd]

> clean up, trying to fix delegation after array reset

- sd-each
	
	目前代理事件只有在 sd-each 中使用，bind 过程直接给delegator （父el） 设置标志，具体过滤器 delegateCheck 的时候不需要执行选择器match的代码
	
- 初始化时机又改了

	原来在绑定的同时调用 update, 现在变成 compile之后 在统一 copy 数据初始化

不断重构，分分合合，跟着作者一起纠结思考~~~

# [01fae38]

> fix delegation, and invoke updates during binding

作者有个习惯，传递中间变量时尽量减少变量的层次，也是对的，避免脑子要hold住太多， 例如把 this.seed.el.delegator => this.seed.delegator，以后修改代码也会稳定点。

这次重构后，初始化和各种 update 逻辑混乱，多次重复，看后面的代码吧

# [832e975]

> separate binding into its own file

现在清晰多了，捋一下

## Seed构造

- 拷贝数据到 scope

- _compileNode
	
	+ parse expression as dirInstance
	+ seed._bind(dirInstance)

		- _createBinding
			
			+ bindInstance.set
			+ collect computed
			+ defineProperty
			
		- dirInstance.bind 
		- dirInstance.update
	
- 处理 _computed

目前重构后思路还是有点绕

# [60e246e]

> clean up binding

# [60a3e46]

> sd-if

只管理了节点增删，应该会有编译之类的逻辑吧？

# [646b790]

> sd-style

直接 el.style[camelProp] = value

# [62a7ebe]

> remove unused

# [76ee306]

> remove redundant dependencies for computed properties

为了解决这个情况

	 scope.total = {get: function () {
        return scope.todos.length
    }}

    scope.completed = {get: function () {
        return scope.total - scope.remaining
    }}
   

提交代码前会出现

completed 依赖 total 和 todos, 之后 todos 更新主动触发 total 和 completed 更新，total的更新再次触发 completed 更新
	
提交代码之后

	completed 依赖 total & remaining
	total 依赖 todo

但是实现逻辑有bug，假如 a 和 b 都依赖 c，那么c 只有能记录一个a / b，谁先 computed 就记录谁

# [f9077cf]

> html and attr directives

# [7fd557c]

> text parser started, features, optional oneway binding for input

# [b5f0227]

> finish text parser

把 {{xx}} 当做 sd-text 处理

# [8c8a07d]

> chinese readme

# [0e91e50]

> minor updates

# [8e028ab]

> remove explorations

作者吸收了 rivets 的精华呀

# [3709862]

update examples

# [9602102]

> update todo

# [e49a31c]

> todo

# [4dad89d]

> readme

# [953fd1a]

> fix array augmentation methods

# [75fc96a]

> license

# [52645e3]

> move defineProperty() into Binding, add setter for computed property

nice

# [e762cc7]

> separate deps-parser

依赖解析的相关逻辑封装，nice

# [7ad304e]

> clean up, add comments

# [c2faf1c]

> parse key path in directive parser

# [d26ec01]

> bower

# [4126f41]

> nested properties


a.b.c

a -> scope = {} -> defineProperty(scope, 'b')

a.b -> scope = {} -> defineProperty(scope, 'c')


# [bf71151]

> update nested props example

# [fa45383]

> shorten some function names since they cant be mangled

# [fa45383]

> shorten some function names since they cant be mangled

为压缩重构变量

另外 依赖变量的命名变得好理解多了，publish & subscribe

# [f5995a5]

> bootstrap returns single seep if that's the only one

纠结了四五回了吧，呵呵

# [5200951]

> restructure todomvc, add $watch/$unwatch

是时候捋一下：

## Seed

- Seed (el, options)

		拷贝 options 到 this
		new Scope 
		拷贝 data 到 scope ( 如果有 )
		invoke ctrl
		call _compileNode
		auto deps extract
		
- _compileNode

		文本节点 {{x}} => _compileTextNode
		
		其他标签 => 
		
		if sd-each
			parse as dir
			seed bind dir
		
		if ctrl but not root
			new chid Seed
		
		else
			each attr
				each exp
					parse as dir
					seed bind dir
			递归 compile child nodes	

- _compileTextNode

- _bind

		identify target seedIns
		call seed._createBinding
		binding <-> dir
		dir.bind // 指令执行前需要预处理，如：初始化变量
		dir.update
		dir.refresh

- _createBinding

- _unbind

- _destroy

## Scope

- $watch
- $unwatch
- $dump
- $serialize
- $destroy

## Directive

- new Directive(dirname, expression, oneway)

		set this._update
		call this.parseKey
		call parseFilter
			

- update
	
- refresh
- apply
- applyFilters
- parseKey


		this.key
		this.arg
		this.inverse
		this.nesting
		this.root

- parseFilter

		this.filters = [{name,apply,args},...]

## deps-parser

- observer
- parse
	
之前我对 filterDeps 的意义误会了，

        <span sd-text="preA"></span>
        <span sd-text="a"></span>
        <span sd-text="getA"></span>
        <span sd-text="getSameA"></span>
 
       scope.preA = 1;
       scope.a = {
        get: function () {
            return scope.preA + 1;
        }
       };

       scope.getA = {
        get: function () {
            return scope.a + 1;
        }
       };

       scope.getSameA = {
        get: function () {
            return scope.a + 1;
        }
       };
       
       setTimeout(()=>{
       	scope.preA = 2;
       })

经过 deps-parser 之后：

	preABind = {
		deps: []//不依赖睡
		subs: [getSameA-textDir, getA-textDir, a-textDir]
	}
	
	getABind = {
		deps: [preABind],
		subs: []//不主动触发谁
		之前误会这里 subs 的缺失，
		这里 subs 意味着所属者更新后会去刷新他 subs 里面的指令，
		我原来设想 preA 触发 a, a 触发 getA... 
		作者的实现摈弃多级 pub，
		直接在依赖树的叶子写入上层，包括根节点的指令
	}
	
	getSameA = {
		deps: [preABind, aBind],// 依赖关系的表明是夸层级的
		subs: []
	}

## Binding

- Binding

		 this.seed = seed
	    this.key  = key
	    var path = key.split('.')
	    this.inspect(utils.getNestedValue(seed.scope, path))
	    this.def(seed.scope, path)
	    this.instances = [] //关联 dirIns
	    this.subs = [] // 关联依赖当前变量的指令，订阅者
	    this.deps = [] // 依赖其他变量的 bindIns

- inspect (value)

>Pre-process a passed in value based on its type

		如果是 {get:, set:} 则标记 computed
		如果是 Array, 则 拦截数组原生方法...
		this.value= value
	
- def
	
	定义 getter/setter

- update (value)
	
		this.inspect(value)
	    each dir.update(value)
	    this.pub()
	
- pub

		each subs : dir.refresh()

# [8411724]

> grunt release task

# [58363eb]

> fix sd-focus, comply with todomvc spec

保证浏览器渲染完dom 之后才调用 .focus

# [8eedfea]

> computed properties now have access to context scope and element

	scope.computedKey = {
		get: function (obj) {
			// obj is
			// { el: self.seed.el,
        	// scope: self.seed.scope }
        	obj.scope.a.b.c
		}
	}
	
	// 调用 get 都会传入这个参数，这样导致深层 scope 访问时
	// 如果没有的话会报错，再自动解析依赖的时候

	function catchDeps (binding) {
	    observer.on('get', function (dep) {
	        binding.deps.push(dep)
	    })
	    binding.value.get({
	        scope: createDummyScope(binding.value.get),
	        el: dummyEl
	    })
	    observer.off('get')
	}
	
	createDummyScope 方法做的就是：
	
	解析 obj.scope.a.b.c 字符串，然后创造
	
	scope = {
		a: noop {
			b: noop {
				c: noop
			}
		}
	}

# [de1c1a1]

> dynamic context computed properties

## deps-parser.js
	
	<li sd-show="a"

	scope.a = {get: function (e) {
        return filters[scope.filter](e.scope.completed)
    }}
	
	aBinding = {
		...,
		contextDeps: ["completed"]// 来自 "e.scope.completed"
	}
	
	seed Of aBinding = {
		...,
		_contextBindings: [aBinding]
	}
	
## Seed

对于形如

	scope.a = {get: function (e) {
        return filters[scope.filter](e.scope.completed)
    }}

因为这里的 obj.scope 是模拟的，所以没有相关的 defineProperty 逻辑，也就是无法自动计算 `a` 的依赖，于是就主动添加到 completedBinding 的 subs

在 deps-parser 处理之后， seed 收集了 _contextBindings 数组
	
	each _contextBindings : seed._bindContexts(binding)


	SeedProto._bindContexts = function (bindings) {
	    var i = bindings.length, j, binding, depKey, dep
	    while (i--) {
	        binding = bindings[i]
	        j = binding.contextDeps.length
	        while (j--) {
	            depKey = binding.contextDeps[j]
	            dep = this._bindings[depKey]
	            dep.subs.push(binding)
	            
	            /*
	            completedBinding = {
	            	...,
	            	subs: [aBinding]
	            }
	            
	            */
	            
	        }
	    }
	}

简单理解，收集 contextDeps 和 deps 的用途差不多

# [a5727bd]

> avoid duplicate context dependencies

# [2d448ea]

> use for loops instead of forEach whenever possible

# [2e5fc62]

> 0.1.4

# [d0c96c5]

> comment updates

# [cfc27d8]

> separate storage in todos example, expose some utils methods

# [8c50a9c]

> 0.1.5

# [cdfe391]

> $load()

# [4772f12]

> fix sd-value and filters.pluralize

# [8a004e7]

> createTextNode in FF requires argument

最新版 chrome 52 用这个方法是需要参数了

# [3c5464c]

> 0.1.6

# [829874e]

> no longer use require() when using dist/seed.js

打包的时候用 wrapper/* 包裹源代码，暴露全局 Seed

# [5bfb4b5]

> stricter check for computed properties

呼应作者那句话『你不限制，真的有人那样用~』

# [2658486]

> add Seed.broadcast() and scope.$on()

# [cc64365]

> put properties on the scope prototype!

# [cc64365]

> put properties on the scope prototype!

- 把 computed 成员的 get/set 方法绑定 this 为 scope
- sd-on 指令触发事件后，参数穿入 seed, 不通过 dir.seed.scope 获得，而是 dir.binding.seed.scope

	绑定指令的时候 dir <-> seed  #1
	
	之后根据变量表达式，找到 target seed，再创建 Binding，此时 bindingIns 关联的 seed 不一定是之前的 // #2
	
	总结：事件处理器关注绑定变量，那么就要找到绑定变量的 target scope,指令 dir 关联 seed, 更关心是在 compile 阶段的哪个 seed（ctrl）下实例化的。还没太明白，为啥不全都关联 target scope?

	

		SeedProto._bind = function (directive) {
	
	    var key = directive.key,
	        seed = directive.seed = this
	        // #1
	
	    ...
	
	    // deal with nesting
	    seed = traceOwnerSeed(directive, seed)
	    // #2
	    var binding = seed._bindings[key] || seed._createBinding(key)
	    
- 从概念上理解，Controller 就是 Scope 的子类，api.controller 的调整，避免不断引用 scope

# [aff965f]

> fix accidental global

刷 commit 吗！

# [d78df31]

> rename internal, optimize memory usage for GC

## 概念名词转变

- Scope -> ViewModel
- seed -> compiler

如果代码写得不优雅，你怎么重构！！呵呵！！

# [a85453c]

> really fix GC

# [a85453c]

> really fix GC

GC 问题经常被忽略，至少成员置 null 得做到。

# [f071f87]

> binding should only have access to compiler

Bind 不管 vm，只管 compiler

# [761b643]

> new api WIP

compiler <-> vm

	捋一下概念：
	我们要实现一个业务模型，接着要写 html, 绑定 vm,
	
ViewModel.extend => new ViewModel => new Compiler

# [c98c8a6]

> working, but with memory issue;

奇淫技巧

# [dddb255]

> new api fixed

# [253c26d]

> 0.2.0

jshint 的 loopfunc 检查还是很靠谱的

# [e9f2223]

> 0.2.1, remove $on/$off

# [174a19f]

> fix dump


# [bf01a14]

> support nested VMs and update examples to use new API

- sd-controller => sd-viewmodel

# [6f0eca4]

> simply api;

# [bce2db6]

> todo

# [0afd700]

> todos

# [e028262]

> watcher

监听 对象/数组 的深层变化， 酷

# [b89092b]

> external observe

# [84538d6]

> wip

defineProperty 相关逻辑从 binding.js 移到 compiler，watchArray 等相关『侦测』代码封装到 observe.js

# [84538d6]

> wip

# [ea792ba]

> make dep parsing work

依赖侦测的时候， computed / __observer__ 的成员不纳入

- 对于 __observer__ 的，你肯定是依赖 它下面的成员
- 对于 computed 的，你也是依赖 get 方法体内使用到的成员

# [d8644b8]

> use standalone build

# [beded7d]

> add asyn compilation ($wait/$ready)

延迟compile时机，例如可以在 init 内 $wait, 然后ajax 得到数据再 $ready

# [0f56b85]

> todo

# [caed31f]

> unobserve when setting new values

# [d2779aa]

> templates

看到一点组件化的铺垫了

# [bc84435]

> wip

- sd-each: sub vmIns 不管理 index
- vmIns.$set 方法，用于，如：sd-checked="a.b"

# [6ec70eb]

> trying to fix nested props

- compiler: 把 observables (非 {get: xxx} 的对象成员)收集，作 observe
- deps-parser.js: filterDeps 不需要了， 因为 emit get 的时候跳过 computed 的

到 0.3 再捋一下

# [5b96bdc]

> small cleanups

# [75a4850]

> allow multiple VMs sharing one data object

场景 var data = {a:xx}

data 用在两个 VM 中，当第二个 vm 在作 observe 的时候， `data. __observer__` 就存在了，虽然作了监控，但是第一次 observe 会对每个 key 触发 set 事件，下面代码相当于再触发一次 set 事件

	if (alreadyConverted) {
                emitSet(obj, obj.__observer__)
            }

# [0c3d629]

> small fix

# [ee09149]

> jshint pass

# [0a45bb8]

> small cleanups

# [77ce1fc]

> directly use array length

把数组长度当做一个成员并且监听，实现不太优雅

数组和对象都看做 observable 就行，多弄一个 arrays 来收集，不够抽象呀！

# [c6c5fdb]

> fix each.js for new architecture

英雄所见略同！

- 把非 {get:xx} 对象 和 数组 都看做 observable, 一起收集
- sd-each: 移除 updateIndexs 逻辑，使用数组 splice 方法，在创建插入时就是准确的 index

splice 拦截里面的 `index` 计算，终于是对了，之前我想了半个小时，坑呀！

# [fbe6b56]

> fix todos example

# [9c4b62f]

> fix init value for Directives

疑问：

	DirProto.update = function (value, init) {

update 方法第二个参数，为啥 comile 初始化值的时候没有设置为 true，啥时候用的

# [e1ce623]

> add utils.extend

顺手移除了 oneway，目测不会放弃 oneway，后面应该会换个更方便的语法

# [9d0d211]

> expression parsing

思路就是把表达式解析成一个 {get:xx} 赋值给 binding.value, 后续就跟之前的 computed 一样处理

# [b379274]

> make it cleaner


# [874afe2]

> $watch/$unwatch

之前的 $watch 是找到 binding[key].subs，然后塞一个 callback 进去，当 refresh 的时候就会调用到这个 callback，非常不优雅，如今的 $watch 相当于一个糖果api，只是帮你监听 change:key 事件

# [c6a25eb]

> npm, remove wrapper

要不要那么纠结！

# [81324ab]

> 0.3.1

# [b5bcee5]

> remove legacy eventbus

这个全局 eventbus 交给用户自己创建更优雅

# [d7f753e]

> comments

学英文~~

# [8eb3c17]

> more meta

# [c6903e0]

> get ready for tests

没人会给没测试的项目 pr，听说 vue 是 100% 测试覆盖率，^_^

# [498778e]

> 0.3.2 - make it actually work for Browserify

捋一把


	vm 		
	|-$el	
	|-$parent
	|-$compiler	<<=>>
			|- el
			|- vm	<<=>>
			|- directives [dir,...]
							|- vm	<<=>> $compiler.vm
							|- el
							|- binding
							|- compiler	<<=>> $compiler
							|- directiveName
							|- expression //完整
							|- rawKey	//除去过滤器
							|- arg	// arg:key
							|- key
							|- nesting (Num)// ^ 往上多少级父亲
							|- root (Bool)
							|- filters [{name:x,apply:x,args}...]

			|- expressions [binding] // 表达式
			|- observables [binding] // 非 {get:xx} 的对象 和 数组
			|- computed [binding] // 表达式 or {get:xx}成员
			|- ctxBindings [binding] // {get:fun} fun中有依赖的
			|- parentCompiler
			|- bindings [binding]
							|- value
							|- isComputed
							|- rawGet
							|- contextDeps ['vm.x',...]// get 中依赖的别的成员
			|- rootCompiler
			|- observer (Emitter)
					|- proxies {}

---
	compiler.bindings = {
		key: {
			instances: [dir, ...]
		}
		subs: [bindingA, ...]
	}
	
	compiler 1...N	(key 1..1 binding) 1..N dir
						===> 值改变之后的更新线路 ===>

---
	
	依赖关系是用 binding 的引用
	
	binding.update()
		each dir in instances : dir.update()
		call pub()
			==>
			each binding in subs : dir.refresh()
										==>
										each dir in instances : dir.refresh()
	
	外部引起的值改变先是 update 再触发依赖自己的 computed 成员去 refresh
---

## ViewModel

- ViewModel (options)

	=> new Compiler(this, options)

- $set (key, value)

	对 vm 设值，key 可以为 a.b.c 这样的

- $get (key)
- $watch / $unwatch
- getTargetVM

	在调用 $set/$get 的时候，前提是有了 basekey 相关的 binding 才行

> vm 是面向开发者的接口，无非就是 set/get/watch，其他逻辑都封装在 compiler 里面

## Compiler

- Compiler (vm, options)
	
		1. extend(this, options)
		2. extend(vm, options.data)
		3. determine el
		4. prototypal inheritance of bindings
		5. call options.init
		6. for key in vm : createBinding(key)
		7. compileNode
		8. for bindIns in observables : 
			Observer.observe(bindIns.value, bindIns.key, this.observer)
		9. DepsParser.parse(computed)
		10. bindContexts(ctxBindings)
			

- setupObserver

 > 反正代码看久了，觉得内部也基于事件的话，理解起来很自然

		observer
	        .on 'get'
	        	depsOb.emit 'get' //为了依赖侦测
	        .on 'set' 
				emit 'change:key'
				bindings[key].update(val)
	        .on 'mutate' 
	            emit 'change:key'
	            bindings[key].pub()

- createBinding

		表达式
		ExpParser.parseGetter(key, this)
		binding.value = { get: getter }
		this.markComputed(binding)
		this.expressions.push(binding)
       
---
		非表达式
		compiler.bindings[key] = new Binding(compiler, key)
		对形如 `a.b.c` 这类，会保证
		binding = {
			'a': 
			'a.b':
			'a.b.c': 
		}
		每层 path 都有一个 bindingIns
		只对第一层 path 调用 this.define(a, aKeyBinding)
		
疑问：为什么要保证每层 path 都有 bingingIns，后面阅读留一下

- compileNode (node, isRoot)
	
	    //if text node 则 compileTextNode(node)
	    //if 标签元素
	        //if sd-each 
	            parse then bindDirective
	        //if sd-vm 且 非根
	                new ChildVM
	        //if 其他元素
                // 遍历属性，跳过有 vm 声明的
                		//遍历表达式
						parse then bindDirective 
	            // 递归 compile 子元素

- bindDirective

		找到 target compiler
		创建 binding
		设置 subs
		执行 开发者的 bind hook 
		if computed 
			call refresh
		else
			call update

- markComputed
	
- define (key, binding)

		针对对根成员，定义 getter/setter，另外 observables 也是在这里收集的
	
		难点：
		
		getter 会触发 'get' 事件，目前是为了依赖侦测，为了获得最『纯净』的底层依赖，
		对以下类型不触发，因为以下类型的值肯定依赖更深的属性：
			isComputed
			value.__observer__
			array 
			
		setter
		对于 computed 的，有 set 方法就直接用，没有就不管了
		非 computed 的话，要先移除 observe 设置之后重新 observe
			解析：primite 类型不痛不痒，对象类型确实是要重新构建 __values__ 之类的

- bindContexts

	subs 存放的是 bindingIns, 而 contextDeps[] 放的是变量名，要通过...
	
- destroy 

	有个细节容易忽略：
	
		<parentCompiler>
		    <subCompiler sd-x="^name">
		</parentCompiler>
		
		subCompiler = {
		    directives: [nameDir]
		}
		
		parentCompiler={
		    bindings: {
		        // 对应 ^name
		        name: {
		        	// subCompiler 销毁时要移除 nameDir
		            instances: [nameDir, ...]
		        }
		    }
		}
	
		dir <==> compiler 这是在编译阶段确定，但是 dir 的执行是跟绑定变量有关的
		参考上面的层级，当 subCopiler 销毁的时候， nameDir 就再也不用了
	
---
	
	表达式的 binding 不存放在 bindings{} 里
---

- compileTextNode

## Directive

- Directive (directiveName, expression)

		each definition
			this._unbind = 
			this._update
			this.xxx = xxx ...
		parse key
		parse filters

- update
	
	值改变的时候会调用，只针对非 computed

	注释说 this will only be called once during initialization 是不对的

- refresh

	值改变的时候会调用，只针对 computed 成员，当所依赖发生改变时

- _update

	开发者扩展

- apply

	apply filter and call _update

- unbind
- _unbind
	
	开发者扩展
	
## Binding

## ExpParser

利用 arttemplate 的compile解析引擎抽取表达式中的变量，然后构造一个 Function，方法体包含这些变量的使用，mock 一个 {get: } 的 computed 成员，有点黑魔法的感觉

## observer

	
	vm.a = { // objA 
		b: { // objA.b
			c: 'this is c'
		},
		b2: { // objA.b2 
			c2: 'this is c2'
		}
	}
	
	objA.__observer__.on 'evName' ()=> {
		compiler.observer.emiit 'evName'
	}
	objA.__values__ = {}
	
	bind (obj, key, path, observer)
	
	>>> bind (objA, 'b', null, objA.__observer__)
		
		objA.__values__[b] <= objA.b
		objA.__observer__.emit 'set' with key 'a.b'
	
		defineProperty objA, 'b', 
			get:
				emit when primite value with full key
				return  objA.__values__['b']
			set:
				objA.__values__['b'] = newValue
				***
				watch (newB, 'b', objA.__observer__)
				***
					===>
					>>> bind (newB, 'b.c', 'b', objA.__observer__)
						newB.__values__['b.c'] = newC
						...
						***
						watch (newC, 'b.c', objA.__observer__)
						***
						exit when primite
	

# [df21257]

> unit tests for binding.js

根据方法分组, 百分百覆盖的借助

。。。测试。。。

# [75dcb03]

> use __proto__ interception for array methods

IE 11 才开始支持呢

# [c94ff6b]

> simplify template API

于是 template 的用法就跟 Backbone 差不多了

# [5ff47a8]

> fix observer mechanism

observable 那类对象，监听的时候会触发各层 'set' 事件，当你通过 vm.a.b.c 设置值的时候，会再触发一次，而且只需要遍历 vm.a.__observer__.values 就行，里面存放的key都是打平的，之前 emitSet 里面递归是不对的

# [4209e27]

sd-if and minor fixes

# [4003fe2]

> implement new API per spec

	function templateToFragment (template) {
	    if (template.charAt(0) === '#') {
	        var templateNode = document.querySelector(template)
	        if (!templateNode) return
	        template = templateNode.innerHTML
	    }
	    var node = document.createElement('div'),
	        frag = document.createDocumentFragment(),
	        child
	    node.innerHTML = template.trim()
	    /* jshint boss: true */
	    while (child = node.firstChild) {
	        frag.appendChild(child)
	    }
	    return frag
	}
	
	node 是个临时的 div 节点，因为没有 frag.innerHTML 

# [8a94192]

> new init/extend options API

# [9297042]

> directive interface change

外部关联 => 构造参数

# [eef0dc6]

> private directives and filters

	parseFilter(filterExps[i],this.vm.constructor.options)
   
   感觉还是要外部穿入 privateFilters {} 更优雅

# [ec86b9f]

> ViewModel.extend() should also extend Object options

# [ec86b9f]

> ViewModel.extend() should also extend Object options

`el` 不继承，props 中，不覆盖原型地继承

# [dbcd78f]

> fix: each options should be mixed into compiler instance

# [55ec2fc]

> fix nested VM example

# [2232cf2]

> $index for each items

# [331f03b]

> array methods should be inenumerable

# [98d1108]

> detach container before batch DOM updates for sd-each

# [61e897e]

> avoid no parent detach

# [1e827e8]

> compiler clean up, vm & partial API, jshint test files

# [61bddb0]

> clean up utils

# [a10fdbd]

> should remove partial attribute

# [b0dcfd5]

> clean up compiler + comments

# [a21e890]

> implement $ event methods, optimize for minification

event scope <=> compiler

# [f1179ac]

> tests for $ event methods

# [3cb7d1a]

> rename sd-each to sd-repeat

# [3159c9a]

> fix sd-repeat pop/shift/splice on empty array

# [9405ed7]

> rename `props` options to `proto`

# [65faa95]

> move all API methods to Seed

# [99b25b2]

rename `prop` option to `scope`

# [af342b7]

> remove context binding related stuff

推荐都用表达式?!

# [094a1fe]

> simple no-data directive

对于 simple dir, 在解析完 compiler.bind 的时候，就只执行一下definition 方法

# [5fa908d]

> sd-id

功能皆指令

# [fb6b4ea]

> fix sd-repeat + sd-viewmodel

不认同这种方案，原来问题：sd-repeat 的每个 item 的 exp 一样，所以代理事件处理器的 key 一样，一旦第一个 item 注册后，这个处理器就有了，后面的 item 同样用这个处理器导致 vm 每次都是一样

这次 fix 通过不代理事件来解决，我觉得还是要代理，vm 定位准确就行了，如果担心 $index 维护麻烦, 可以加上更准确的随机 id, 这样每个item能够准确定位到  vm

# [db22a2a]

> fix dependency tracking for nested values

"item.title + msg" 解析除 vars=['item', 'msg'], 而实际要 ['item.title', 'msg'] 这种 path

# [91d8528]

> use Object.create(null) for better hash

get技能：

a = {}
a.constructor // Object
a 具有 Object.prototype 的所有方法
...

b = Object.create(null)
b.xx //啥都没

# [f4d42cc]

> add code coverage for unit tests

用 jscoverage 作测试覆盖率监测，原理就是将 all.js 解析插入生成一份带标记的代码 all-cov.js，功能跟原来一样，然后在 runner.html 引入这份 all-cov.js

# [db284ff]

> move $index to VM instead of item, add $parent & $collection

这样直接在 compiler 上 observe $index 方便多了

# [11f89fb]

> fix transition logic so previous unfinished transitions can be properly cancelled

处理动画和class关系，比 ionic 的 actionsheet 部分优雅多了

# [257781c]

> implement custom javascript transitions

*****cool~

# [9dc45ea]

> sd-on can now execute expressions

支持这种             
	
	sd-on="click: this.a = 'a'"

内部把表达式包装成方法

# [0f7a929]

> fix observer emitSet() when observed by multiple vms

疑惑：改完后的效果也是一样的，按照现有的改法，怀疑是不是要把下面这段干掉

	ob
    .on('get', proxies.get)
    .on('set', proxies.set)
    .on('mutate', proxies.mutate)

# [a8e12f8]

> Firebase example + bug fixes
> 
> - move ensurePath() to Observer.js
> - add Observer.ensurePaths(), which deals with the situation
> when a scope object with nested values is given a new value
> with incomplete nested structure. e.g. this.val = {}
> - fix Directive argument regex
> - sd-model: no longer locks during set() so filters work on
> input fields
> - sd-repeat: handles undefined update, also fix transition
> - utils.attr now takes additional `noRemove` argument
> - ViewModel.$emit now also triggers event on itself

下面这段改动比较难理解

		     observer
	         .on('set'
	         ....
	+            check(key)
	               ...
	         })

有些 {$get:} 中依赖了还没有创建绑定的key，在依赖侦测触发 get 的时候就需要动态创建

# [3eba564]

> fix sd-model selection position issue

sd-model 内部的 set 方法是为了改变 vm 中的模型值，其中的 update 方法是 模型值 => dom 值，所以由浏览器事件处理器 set 方法执行后，dom值无需考虑，只需要设置 vm 的值，于是就 lock 住 update 方法的执行，考虑到 filters 后的值就不是浏览器 dom 本身的值（例如checkbox的checked），于是要执行 update 方法，值为 filter 之后的值

# [c8779f1]

> remove dead code

之前 contextDeps 相关代码移除的时候，又给疑惑没解决

移除那部分逻辑之前

            var Demo = Seed.extend({
                lazy: true,
                init: function () {
                    this.a = {c:1};
                },
                proto: {

                    d: {
                        $get: function () {
                            return this.msg +(this.a.c || '')
                        }
                    }
                }
            })

因为 this.a = {c:1} 在解析依赖之前会 observe(x, 'a') 之后保证了 compiler.bindings['a.c'] 存在，然后 解析 d.$get 就靠谱了

可是，如果是 this.a = {}，那么在依赖侦测 d.$get 之前就不会有 compiler.bindings['a.c'], 相关依赖就不会建立，

之前移除了 contenxBinding 逻辑的时候就有这个疑问，怎么破？默认都要求现有初始化值？

# [bc0fd37]

> New ExpParser implementation
> 
> - extract all paths and replace them with correct reference
> - in the process, create missing bindings on the owner vm's compiler

sd-text="a.b"

从当前递归往上找到拥有 `a` 的 vm， 作为目标 vm，另外 当目标 vm.$compiler.bindings 没有 `a.b` 则会创建一个，解决了上面的疑问！

# [795d6b9]

> conditional dependency tracking

对于

    <p>{{ok ? yesMsg : noMsg}}</p>

原来

	=> function () { return this.ok ? this.yesMsg : this.noMsg}

经过 fix 之后

	=> function () {this.ok;this.yesMsg;this.noMsg;
	return this.ok ? this.yesMsg : this.noMsg}

# [19d15ec]

> Improvements to observed array extension methods

get 技能:

	取整
	a=2.12312
	~~a  //=> 2


# [628c42c]

> component refactor

自定义标签 elements 重构成 v-component-id

# [216e398]

> observer rewrite

# [dc17a4e]

> add benchmark for todomvc example

get 技能：

	log 同步异步渲染时间
	function report () {
		sync = now() - start
		setTimeout(function () {
			async = now() - start
			console.log('render: ' + render.toFixed(2) + 'ms')
			console.log('sync:   ' + sync.toFixed(2) + 'ms')
			console.log('async:  ' + async.toFixed(2) + 'ms')
		}, 0)   
	}

# [14d8ce2]

>compiler rewrite - observe scope directly and proxy access through vm

- vm.$scope => scope
- 拷贝了 scope 到 vm 之后，直接 observe(scope, '',compiler.observer)
- 每层对象都有自己独立的 `__observer__` 和 `__values__`
	
		scope 对应 compiler.observer 收到 'a.b.c' 事件
	
		scope.a = { a__observer__.emit('b.c', 'strc')
			b: { b__observer__.emit('c', 'strc')
				c:'strc'
			}
		}
	
# [f4861ca]
> npmignore

7.0 捋一下

## func

- config (opts) 如：更新前缀

- directive (id, fn) 注册指令 def

- filter (id, fn) 注册指令 def

- component (id, Ctor) 
>注册组件，用 util.toConstructor 转为 ViewModel(或子类)

- partial (id, partial)
>注册模板，用utils.toFragment 转为 frag 节点

- transition (id, transition) 
>注册动画 def

- extend (options)

        /*
            // this as parent vm

            // inherit options from  parent vm's options

            // process options

            // def a fun as ctor, calling super ctor inner

            // inherit proto, sub.vm.proto <= parent.vm.proto

            // spec `constructor` for sub proto

            // copy `methods` to vm.proto

            // allow extended VM to be further extended
        */

- inheritOptions (child, parent, topLevel)
>对象则 if 拷贝(一层)，方法则 mergeHook，忽略 `el` `methods`

- mergeHook (fn, parentFn)

- updatePrefix () 
>更新 `v-xx` 中的 `v`

## Compiler

### props

+ options
+ data
+ vm
+ dirs []
+ computed []
+ childCompilers []
+ emitter <Emitter>
+ bindings []
+ parentCompiler
+ rootCompiler
+ childId
+ bindings {}
+ observer <Emitter>
    - proxies
+ [repeatIndex]

### func

- Compiler (vm, options) {

	    // flag initing
	
	    // process and extend options
	
	    // copy data, methods & compiler options
	
	    // setup element
	
	    // set compiler properties
	
	    // inherit parent bindings
	
	    // set inenumerable VM properties
	
	    // set parent VM
	    // and register child id on parent
	
	    // setup observer
	
	    // beforeCompile hook
	
	    // the user might have set some props on the vm 
	    // so copy it back to the data...
	
	    // observe the data
	    Observer.observe(data, '', compiler.observer)
	        //compiler.observer emit ''+'a.b.c'
	        data = {// data.__ob__ emit 'a.b.c'
	            a : {// data.a.__ob__ emit 'b.c'
	                b : {//data.a.b.__ob__ emit 'c'
	                    c: 'this is str c'
	    
	    // for repeated items, create an `$index` binding
	    // which should be inenumerable but configurable
	
	    // vm `$data` getter/setter
	
	    // now parse the DOM
	        // create necessary bindings
	        // and bind the parsed directives
	
	    // extract dependencies for computed properties
	    //  set this.binding.deps  dep.binding.subs
	
	    // flag end init
	
	    // post compile / ready hook

- setupElement (options) 

		// ensure node, handler opt.template, apply el props/attrs

- setupObserver () // expose evt


- compile (node, root) {

	    //如果是 标签节点
	
	        // if repeat
	
	            //parse then bind
	
	        // if v-component but not root
	
	            // parse
	            if (directive) {
	                // component directive is a bit different from the others.
	                // when it has no argument, it should be treated as a
	                // simple directive with its key as the argument.
	                if (componentExp.indexOf(':') === -1) {
	                    directive.isSimple = true
	                    directive.arg = directive.key
	                }
	                compiler.bindDirective(directive)
	            }
	            // bind
	
	        // else
	
	            // node.vue_trans <= v-transition
	            // replace innerHTML with partial
	            // compileNode
	
	
	    // 如果是 文本节点
	    compiler.compileTextNode(node)

- compileNode (node)

	    // each attr : each exp : parse then bind
	    // recursively compile childNodes



- compileTextNode (node)

	    // TextParser.parse as tokens
	    // each tokens:
	        //if a binding with key
	            // if '>' as partialId, compileNode()
	            // else parse as sd-text then bind dir
	
	        //if a plain string then createTextNode(token)

- bindDirective (directive)

	    // append to dirs
	
	    // for a simple directive, simply call its bind() or _update()
	    // ** and we're done.
	
	    // otherwise, we got more work to do...
	
	    // if exp
	        // expression bindings, created on current compiler
	
	    // if data or vm has base key
	        // If the directive's compiler's VM has the base key,
	        // it belongs here. Create the binding if it's not created already.
	        // ensure binding for that key
	
	    // else
	        // due to prototypal inheritance of bindings, if a key doesn't exist
	        // on the bindings object, then it doesn't exist in the whole
	        // prototype chain. In this case we create the new binding at the root level.
        
        // 为什么：会出现不在 vm/data 上，却有 compiler.bindings[key] 的情况呢？
        // 答：手贱手动 delete
        // binding = bindings[key] or rootCompiler.createBinding(key)

	    // push dir to binding.instances
	    // directive <--> binding
	
	    // invoke bind hook if exists (dev make it)
	
	    // set initial value
	    // refresh for computed, update for other


- createBinding (key, isExp, isFn)

	    // if exp
	        // a complex expression binding
	        // parse exp to generate an anonymous computed property {$get:xx}
	        // apply `value`
	        // mark computed
	        // push to exps
	
	    // just key
	        // if root key
	            // define getter/setters for it.
	        // else
	            // ensure path in data so it can be observed
	            // ensurePath for `compiler.data`
	            // ensure binding for parentKey (recursively)

- define (key, binding) 

> key setter/getter on `data`

	    // binding.value = data[key] // save the value before redefinening it
	
	    // if like {$get:x} then process markComputed
	
	    // ensure data[key] = undefined if non-exist
	
	    // if the data object is already observed, that means
    
	    /*
	    疑问：在 Observe.observe 的时候，会执行一次 convert，并且触发 set
	    然后进入 check(key)，只有又进入 define, 这里又执行一次 convert
	
	    多余了，看 todo demo 里面，未做任何操作清空下，set 事件两次了
	     */
	    
	    // this binding is created late. we need to observe it now.
	    // if (data.__observer__) {
	    //     Observer.convert(data, key)
	    // }

- markComputed (binding)
   
	    // flag isComputed 
	    // bind the accessors to the vm
	    // this.computed add binding

+ getOption = function (type, id) {
   
		 // 递归查找 [this/parent].options[type][id]

+ execHook (id, alt)

- destroy ()

	    // compiler.execHook('beforeDestroy')
	
	    // unwatch
	    // observer.off()
	    // emitter.off()
	
	    // unbind all direcitves
	        // if this directive is an instance of an external binding
	        // e.g. a directive that refers to a variable on the parent VM
	        // we need to remove it from that binding's instances
	
	    // unbind all expressions (anonymous bindings)
	
	    // unbind/unobserve all own bindings
	
	    // remove self from parentCompiler
	
	    // finally remove dom element
	
	    // compiler.execHook('afterDestroy')

- getRoot (compiler)

## ViewModel

## props

+ $ {childId:<VM>,...}
+ $el
+ $compiler
+ $root <VM>
+ $parent <VM>
- func

## func

- ViewModel (options) 直接 new Compiler
- $set (fullkey, value) 找到 basekey 对应 vm 然后设置
- $watch (fullkey, callback)
- $unwatch (key, callback)
- $destroy ()
- $broadcast () 递归向下发事件
- $emit () 自身&往上递归发事件
- $on ()
- $off ()
- $once ()
- $appendTo (target, cb) 
- $remove (cb) 
- $before (target, cb)
- $after (target, cb)
- getTargetVM (vm, path) 

		//vm.$compiler.bindings[baseKey].compiler.vm

# Directive

## props

+ compiler
+ vm
+ el
+ isSimple
+ expression
+ rawKey
+ key
+ arg
+ isExp
+ filters [{name,args,apply}]

## func

+ bind ()
+ _update ()
+ _unbind ()
+ ... copy from definition
+ Directive (definition, expression, rawKey, compiler, node)

	    // spec isSimple
	
	    // mix in properties from the directive definition
	
	    // empty expression, we're done.
	    
	    // this.isSimple
	
	    // parseKey(this, rawKey)
	
	    // this.isExp
	
	    // parse filters

- parseKey (dir, rawKey)

- parseFilter (filter, compiler)
- update (value, init) // 4 non-computed
- refresh (value) // 4 computed
- apply (value) 

		// apply fileter and call this._update
	
- applyFilters (value)
- unbind (isForUpdate)
  
  	  // call this._unbind

- Directive.split (exp)
- Directive.parse (dirname, expression, compiler, node)

	    //check and return new Directive(dir, expression, rawKey, compiler, node)

## Binding

> 每一个 vm 上的属性（路径）都有一个对应的 Binding 对象，这个对象有多个作用在 DOM 上的 指令 实例，以及多个依赖
> binding 跟属性一一对应，所以 update 等方法的触发入口是属性被改变了

## props 

- value
- isExp
- isFn
- root
- compiler
- key
- subs = []
- deps = []
- isComputed //表达式 / {$get:xx}
- instances [<Dir>,...]

## func

- Binding (compiler, key, isExp, isFn)

- update (value)

        // set this.value
        // each dir update
        // pub()

- refresh 
   
	    // each dir refresh
	    // pub()

- pub()

  	  // each sub.refresh

- unbind () {
    
        //unbind dirs
        //remove from other's deps

# Observer

- watchObject (obj) 

	    //=> each key: convert(obj, key)

- watchArray (arr, path) 

  	  	// 重写数组的方法

- isWatchable (obj)

- emitSet (obj)

 	  	 // 触发对象各路径 set 事件，通常用在重复 observe 的时候

- convert (obj, key)

	    //定义 obj.key 的 accessors, 通过 obj.__observer__ emit 事件出去
	    //对 obj[key] observe

- copyPaths (newObj, oldObj) 

    	//保证旧对象的叶子路径都在新对象上

- ensurePath (obj, fullkey)

		// 保证 各层路径 accessed 和 enumerated

- observe (obj, rawPath, observer)

	    // 监听 obj 各层值，通过 observer 对外暴露事件
	    // rawPath 表明 obj 在它的 vm 中的路径

- unobserve (obj, path, observer)

    	//obj.__observer__.off(everyKey, observer.proxies[path][everyKey])


# Filters

## func

- capitalize (value) {

- uppercase (value) {

- lowercase (value) {

- currency (value, args) {

- pluralize (value, args) {

- key (handler, args) {


# Utils

## props

- hash {}
- components {}
- partials {}
- transitions {}

## func

- attr: function (el, type, noRemove)
- defProtected: function (obj, key, val, enumerable, configurable)
- typeOf (obj)
- bind (fn, ctx)
- toText (value) // ensurce string|number|bool
- extend: function (obj, ext, skipIfExist)
- unique (arr) // new refer
- toFragment (template) //ensure return dom (frag)
- toConstructor (obj) //ensure return VM Ctor
- processOptions (options)

	     /* 
	     convert options like {
	          components: [VMCtr, ...],
	          partials: [fragNode, ...],
	          template: fragNode
	     }
	     */

- log (args...)
- warn (args...)
- nextTick (cb)
- makeHash ()

# Deps Parser

## props

- observer <Emitter>

## func

- parse (bindings)

## Exp Parser

- getRel (path, compiler)

> 如果当前没有 baseKey，则递归父级 形成 $parent.$parent.x.x 的访问路径
> 另外会保证在对应 compiler 上存在 binding[path]

- makeGetter (exp, raw)

- escapeDollar 

> Escape a leading dollar

- parse (exp, compiler)

> 将表达式语句解析成 function () {this.fullkeyA;this.fullKeyB;return xxxx}

# Text Parser

## func

- parse (text)

> loop match key like regex

## Transition

- transition (el, stage, cb, compiler)
- applyTransitionClass (el, stage, changeState)

> Togggle a CSS class to trigger transition

- applyTransitionFunctions (el, stage, changeState, functionId, compiler)
- sniffTransitionEndEvent ()

# Emitter
> use nodejs one

# dirctives

- component
- if
- on

	疑问：
		
			if (compiler.repeat &&
		            // do not delegate if the repeat is combined with an extended VM
		            !this.vm.constructor.super &&
		            // blur and focus events do not bubble
		            event !== 'blur' && event !== 'focus') {
	
		
	其中对于 !this.vm.constructor.super 的判断不是特别理解，如果强调 super 存在，那么只有 ViewModel.extend 返回才有 super, 那么是在意 数据有可能是继承过来的？？之前做过解释为什么 repeat 在现有逻辑里面不能代理事件，这里强调了 !super 还没弄清楚原因。

- model
- text
- html
- style
- class
- attr
- visible
- show
- repeat

# configs

> 除了配置 slient/debug， 其他通常不会用到

    prefix      : 'v',
    debug       : false, (console.log)
    silent      : false, (console.warn)
    enterClass  : 'v-enter',
    leaveClass  : 'v-leave',
    attrs       : {}

> 0.7.0 终于撸完，感觉读的没写的快~

# [5c73a37]

> async batch update - first pass

疑问：为什么用 setTimeout 替代 requestAnimationFrame?
帧率太快了用慢一点的 setTimeout flush update 能理解，但是干掉 nextNick 为啥呢？

# [c7b2d9c]

> nextTick phantomjs fix, unit tests for batcher, config() api addition

吐槽一下！

# [7fc537a]

> fix functional tests for batcher

bug：model.js 里，还是需要判断 config.async

# [331bcc6]

> avoid duplicate Observer.convert()

目前为止，我都目测出每次提交的主要不足，喜欢这种跟着作者一起思考，也许思考得更多的，脑缺氧，加油！

# [c7f8a68]

> test using gulp plugins instead

部分工程build任务用 gulp 工具替换，特别是stream 类的任务

# [d8e3853]

> use custom stream functions

why? 为了更快吗，肯定会换回来！

# [1e58721]

> perfectly resolve Chinese input methods issue with composition events

当年看 NG 源码就很惊喜原来有这个事件，莫非~呵呵~

# [212990a]

> make exp-parser deal with cases where strings contain variable names

表达式形如 sd-text='"a"+a' ，会把"a"中的 a 当做变量，解决方案是，先对 `"xx"` 占位，最后恢复

疑问：先占位后恢复的过程，如果两个字符串长度一样，那么`strings[i]` 第二次赋值就会覆盖前一次。

我会用随机数作key，用map缓存这些字符串

# [04249e3]

> Component Change
> 
> - `v-component` now takes only a string value (the component id)
> - `Vue.component()` now also registers the component id as a custom element
> - add new directive: `v-with`, which can be used in combination with `v-component` or standalone

# [4129377]

> use karma for unit tests

捋一下工程构建和持续集成吧

## Tasks

>     grunt.registerTask( 'travis', [
>         'build',
>         'instrument',
>         'karma:phantom',
>         'casper',
>         'coveralls'
>     ])

> 逐个分析每个任务

### build

作者自己写的 `gulp-component` (`component-builder` 的 gulp wrapper吧) 对 component.json 解析，目前来说就是收集所有声明的 js 文件然后合并流。

1. header banner
* out ./dist/vue.js
* print size
* uglify
* header banner
* rename vue.min.js
* out ./dist/vue.min.js
* print size
* gzip
* out ./dist/vue.min.js.gz

### instrument
	
收集所有 vue 代码然后输出 ./test/vue.test.js，目的是为了后面做代码覆盖率，所以没有要求压缩啥的

### karma:phantom

	browsers: ['PhantomJS'],
    reporters: ['progress', 'coverage'],
    preprocessors: {
        'test/vue.test.js': ['coverage']
    },
    coverageReporter: {
        reporters: [
            { type: 'text-summary' }
        ]
    }

在 phantomjs 跑一遍所有单元测试代码，但是在单元测试前有个预处理，就是对 vue.test.js 作 coverage, 实际上就是对 vue 源码的xx分支打上一些标记，然后单元测试的时候使用了这份标记后的源码，得到单元测试的覆盖率， cool~

### casper

相当于命令行执行 casperjs 命令作 e2e 测试 test/functional/specs/ 下的用例

### coverall

在指定目录下生成一些单元测试覆盖率报告（lcov-report），也是便于后面使用 https://coveralls.io/ 显示一些覆盖率之类

# [0dcf28e]

> add saucelabs!

saucelabs.com 是一个云端浏览器测试平台，很酷~

runner.html 配合 saucelabs，添加了

				onload = function () {
                var runner = mocha.run()
                runner.on('end', function () {
                    window.mochaResults = runner.stats
                })
            }

便于 saucelabs 收集错误？

并且本机使用 grunt-sourcelabs 插件直接跟 saucelabs 云端连接进行测试，太爽了，过去苦逼的日子呀~~

# [08ba942]

> fix expressions on repeated items

对于 sd-on， 如果 isFun 的话，对于表达式的绑定，当代理事件执行的时候，call 到 taget el 的 vm，能解决像 {{$index}} 这样的问题问题，不过感觉还是黑魔法

# [5f21eed]

> Fix inline partial parentNode problem
> 
> Directives inside inline partials would get the fragment as the parentNode
> if they are compiled before the partial is appended to the correct parentNode.
> Therefore keep the reference of the fragment's childNodes, append it,
> then compile afterwards fixes the problem.

解决的问题是 compile 的时候 parentNode 不对，我觉得把这种情况的 insert 提前，这样整个 compile 过程还是有序一点

# [1fb885b]

> bindings that are not found are now created on current compiler instead of root

当一个 basekey 不存在当前 vm，之前是在 root 去 createBinding，现在是往上递归找到最近的，拥有这个 key 的 vm（compiler）再去 ...

有个小细节，判断 vm 存不存在，用的是 basekey，createBinding 用的是 path (fullkey)

# [9a45f63]

> clean up get binding owner compiler logic

用 creatObject(null) 代替 `{}`，省了 hasOwnerproperty 的判断，知道 createObject(null) 的时候怎么就没想到省了这部分代码呢~

# [f139f92]

> simplify binding mechanism, remove refresh()

之前捋 update, refresh, pub 的逻辑时就很想帮他重构！！！

# [f139f92]

> simplify binding mechanism, remove refresh()

原来 update 和 refresh 针对 non-computed 和 computed 使用，现在统一为 update，两者不同的是取值调用不同，一个是调用 get 方法，一个直接用 value

# [62fd49a]

> do not modify original computed property getter/setters

保留属性本身的特性（可遍历...），另外更新的时候，computed 类型的 value 不更新，那有什么机制可以更新 computed 类型 value 的 get 方法呢，有没必要？

# [7a6169c]

> separate computed properties into `computed` option

我觉得好处就是：避免了一个潜规则，形如 `{xget:xx}` 这样的就是 computed

# [50a7881]

> refactor Compiler.createBinding

最近几次提交感觉就是一个目的：用户数据不去弄它，这次重构后就很清晰，`data` 不去碰，额外的脏东西就放在 vm 上

捋清:

computed 表示要计算的，binding.value = {xget:xx}, 表达式 和 computed:{} 下的 binding 都会 flag 为 isComputed

表达式的 binding 不放在 bindings 中，存在 exps 数组中

# [9c2bb94]

> common xss protection

exp-parser 过滤了一些关键字等，但是没过滤 constructor, 可以更改某些类的 contructor 来攻击

# [5ea44fb]

> make v-repeat work with primitive values + minor directive refactor

v-repeat 在 buildItem 的时候会把数组元素当做 vm 的数据来用，vm 构造过程把数据当做引用类型来用，所以把 primitive 包装成 {value: xx}，使用的时候要脑补 value 表示啥

同时对数组扩展了一个 set(index, data) 方法

下面例子

	var numbers = [1, 2, 'text']
    var a = new Vue({
        el: '#test',
        data: {
            numbers: numbers
        }
    })
    a.numbers.set(2, {value: 'xxx'})
    
调用了 set 之后，此时内部 collection 中，该位置的元素会变成 {value:'xx'}，因为监听了 'set' 事件，进入下面

	if (primitive) {
	    data.__observer__.on('set', function (key, val) {
	      debugger;
	        if (key === 'value') {
	            col[item.$index] = val
	        }
	    })
	}

会把 'xxx' 替换

整个过程需要一直脑补 `value` 这个 key，有没更优雅？

能否在 mutate 事件中把这个 `value` 拆成 primate？

# [0bee5a0]

> better xss mitigation

- 浏览器处理 html 的时候，先把源码 unicode 转化，所以说表达式中可以通过把危险代码转化为 unicode 执行

- 加强 `.constructor` 检测，防止形如 Array.prototype['c'+'onstructor']

# [ce4342b]

> revert repeated item $destroy behavior

是考虑到几乎不会有从 item 的 vm 主动 $destroy 的情况？

# [f2e32ab]

> Make Observer emit `set` for all parent objects too

这样有问题：如果深层属性改了，那么 updateBinding xdata 多次

# [77ffd53]

> add tests for object outputing + avoid duplicate events

忽略了compile 过程 emitSet，还是会 updateBinding 多次，捋一下用例


	 var a = new Vue({
                el: '#hi',
                data: {
                    address: {
                        city: 'sz'
                    }
	...
	

	
	$data = {	 //$compiler.observer
		
		address: { // a__observer__
			
			city: 'sz' 
			
		}
	
	}

	a.$data.address = {city: 'gz'}

分析一下 改变 $data.adress 之后的事件流
	
	// 从 convert 中的 setter 中触发
	1. $compiler.observer.emit ('set', 'address')
	// 此时 updateBinding 一次 key 为 'address'
	// 之后会 observe({city:gz}, address, a__observer__)
	// 这个过程会触发 emitSet

	2. 从 a_observer__ 这次触发 'set' key 为 'city'
		3. proxies chain 到 $compiler.observer 触发 'set' key 为 'adderss.city'，此时 rawPath

	总结前后两次 updateBinding key 分别为 'address' 和 'address.city'

如果

	a.$data.address.city = 'xx'

	a__observer__.emit set 'city' (propagate=true)
		=> $compiler.observer.proxies['address'] 处理器
			$compiler.observer.emit set 'address.'+'city'
			propagate: $compiler.observer.emit set 'address'

总结前后两次 updateBinding key 分别为 'address.city' 和 'address'

好乱咯~~

# [393a4e2]

> v-repeat object first pass, value -> $value

脑补不出 v-repeat 传 object 的场景

# [e749452]

> fix v-repeat reuse vm insertion when detached by v-if

v-if 的设计本身就很巧妙，处理 v-repeat 的时候，太巧妙了！

回忆：

v-repeat 插入 item 的 el 时，会找到一个 ref 作为参照，插到他前面，可以是个元素，或者一个 hack 的 comment 节点

考虑到有的 item 的 el 处于 v-if (false)，不过不要紧，v-if 为其 el 提供 el.vue_ref 给外部用于参照，v-if 的渲染逻辑也是用到这个 vue_ref 作参照的

当复用 item 的时候，发现该 item 也是 v-if(false)，那么就是直接把当前 item.el.vue_ref 参照 ref 插入到恰当位置

# [0e486a3]

> repeat object sync and tests

在每个数组扩展方法区加 updateObject，不如对 arrayproxy 本身增加扩展机制，增加 hooks，解耦

看完这次修改，感觉 v-repeat 用在对象也是很酷，特别是 sync 到对象的时候觉得很酷

# [31f3d65]

> sync back inline input value if initial data is undefined

这个体验很重要，例如你 checkbox 在 html 本来有默认值，你 bind 之后，因为 data 对应部分为 undefined，可能跟默认值冲突，那么当然保留默认值啦~

对 v-model 来说，update 就是 data（vm） 引起的改变然后用 update 方法改变 dom 的某些属性，_set 就是 v-model 内部从 dom 到更新 vm 的方法，所以当初始化 data 对应部分为 undefined，直接执行 _set，意味着就用 dom 本身的值，去重置被初始值 undefined 污染的 vata（vm）

# [35967cb]

> vm.$set no longer need to check owner VM at run time

为啥 ownerVm 不一定是 dir.vm，之前提过，把 vm 看作 data 的话，data 是跟  binding 关联的，所以说跟 target vm is target data, so then target vm is target (data)binding.compiler.vm

# [60e5154]

> v-on delegation refactor, functional tests pass

	
	<div v-on="click:onClick"></div>
	
	{
		delegators: {
			click: {
				targets: [{
					el: div,
					hanlder: onClick
				}],
				handler: {
					loop check target
						exec target.handler when mach
				}
			}
		}
	}

# [bda8e76]

> js transitions => v-effect

动画可以用 animation 或者 transform 来做，现在 v-animation / v-transition 为了区别使用那种动画结束事件，v-effect 用于自定义动画入场转场回调

略显麻烦，能否监听两种事件，处理器为 once call ？这样使用者无需关心这个细节

# [63e9d72]

> wrap setTimeout in JavaScript effect functions

在自定义动画方法里暴露一个 "setTimeout" 方法，该方法会收集该转场期间的所有 setTimeout，便于重复动画时清空上次转场遗留的未完成的

# [1a05dea]

> v-with allows linking keys between child and parent VMs

我觉得顺手解决了一个 bug

如果元素本身有 vm，那么直接在上面 createBinding，比起原来只依据 isEmpty 是否来创建 vm 靠谱

# [5100f3e]

> setTimeout(0) is faster than rAF

这个有点牵强，闲时确实靠谱，但是 rAF 是绘制时机响应，//留意

# [5331df5]

> clean up v-repeat buildItem()

捋一把 if、repeat

跟之前没差，变量名命名语义化一点而已

# [0893163]

> add `parent` option

把 {compilerOptions:{parnetCompiler:xx}} 放到外面
{parent:parentVm, compilerOptions:{}}

处理器来更加面向数据模型，不用考虑太多散状的状态引用

# [95c1c16]

> destroy children

想起多年前更别人正直的命名规范，我依然认为 node.childrens 比起 node.childrenNodes 靠谱多了

# [c6961ae]

> vm.$options

作者一直把很多概念/脏东西 屏蔽掉，尽量暴露 vm 让开发者理解使用，酷~

# [010cda6]

> array linking rough pass

arr = [obj] 现在 obj 改变触发 set 事件也会传到 arr 了，太脏了，呵呵

- convert 准备对象的 __emitter__，把 set 事件传递到所在各个数组
- convertKey: 对指定对象下的 key 定义 get/set

# [5fb9f24]

> still need to check conversion in v-repeat update

现在的代码命名好多了，好理解很多，也许是一路跟过来的关系吧

# [bedcb22]

> allow nested path for computed properties that return objects

        
   {{nested.value.a}}

    computed: {
        nested: function () {
            return {
                value: {
                    a: this.a + 2
                }
            }
        }
    }

修改前，对于这样的例子是无力的，在原来的逻辑里 createBinding 的时候会进入

	// ensure path in data so it can be observed
    Observer.ensurePath(compiler.data, key)
    var parentKey = key.slice(0, key.lastIndexOf('.'))
    if (!bindings[parentKey]) {
        // this is a nested value binding, but the binding for its parent
        // has not been created yet. We better create that one too.
        compiler.createBinding(parentKey)
    }

等同于把 nested 当作纯字面对象来理解，这样 ensurePath 之后，初始值就是 undefined, 除非再次手动赋值，否则这个绑定变量不会有值

这个修改之后，把 nested 当作表达式，每次都会去调用，当然就能返回正确之

# [d56feb0]

> enable interpolcation in literal directives except v-component

ref 和 partial 都解耦成指令了

# [4ebff99]

> computed filters first pass

表达式解析多了一个参数 filters, 对表达式的值在 wrap 一层方法，递归执行 filters

对于 filters: {x} 中的方法如果方法体有 `this.` 那么这个绑定会当做表达式来解析

# [3df6343]

> v-if refactor + change to use childVM

v-if 不能跟 v-repeat 和 v-view 一起使用

# [5e638a0]

> methods

对 methods: {} createBinding 之后，subVm 直接 get 父 vm 的方法也行了

# [224d67d]

> fix #520 let watchers handle callbacks that trigger removeCb()

那位置调换了呢？还是在一次执行周期打上标记，执行过就标记一下

# [cf37f7e]

> Release-v0.10.6
	
	src
    |- batcher.js
    	
    	异步批处理任务
    
    |- binding.js
    	
    	可以理解为表示一个 {{xx}} 的实例，这个绑定变量下会有多个指令
    
    |- compiler.js
    	
    	编辑咯
    
    |- config.js
    	
    	诸如前缀 'v-' 等配置
    
    |- deps-parser.js
    	
    	依赖侦测，建立 binding 之间的依赖
    
    |- directive.js
    
    	表示一个绑定的行为，内部包含过滤器
    
    |- directives
        |- html.js
        |- if.js
        |- index.js
        |- model.js
        |- on.js
        |- partial.js
        |- repeat.js
        |- style.js
        |- view.js
        |- with.js
    |- emitter.js
    	
    	like nodejs' emitter
    
    |- exp-parser.js
    
    	解析表达式，构造一个对表达式求值的方法
    
    |- filters.js
    
    	内置过滤器
    
    |- fragment.js
    
    	html string -> fragment，处理 tr 等特殊情况
    
    |- main.js
	
		ViewModel 暴露公共方法
    
    |- observer.js
    	
    	对象/数组 监测机制
    
    |- template-parser.js
    	
    	将所有可能代表一个template的字符串 convert 成 fragment
    
    |- text-parser.js
    	
    	解析包含绑定的文本（文本节点，属性值, ...）
    
    |- transition.js
    	
    	动画相关，能处理 animation/transition/custom func
    
    |- utils.js
    	
    	工具
    	
    |- viewmodel.js
    
    	管理数据状态，代理了事件、dom 接口
 

疑惑：
	
	if (params && params.indexOf(attrname) > -1) {
	    // a param attribute... we should use the parent binding
	    // to avoid circular updates like size={{size}}
	    this.bindDirective(directive, this.parent)
	} else {
	    this.bindDirective(directive)
	}

		
如果指定 paramAttributes 有 'size'，那么 binding update 的时候会去更新 vm.size, vm.size 改变引起 binding update，为了不要死循环

	bind 'size' on parentCompiler
	-- update -->
	child.vm.size = newValue

依据这种机制，那么 paramAttributes 就是用来单向传递参数？

而 v-with 是用来双向传递参数？

*get 技能*

	str.replace(ESCAPE_RE, '\\$&')
	// $& 表示整个匹配内容

# after 0.10.6
> 开始重写

# [0b70e52]

> exploring scope inehritance

*get 技能*

	
	parent = {}
	def(parent, 'name', {set: ()=> {console.log('set in parent')}})
	
	Child = function (){}
	Child.prototype = parent
	
	child = new Child()
	
	child.name = 'peter'
	// 打印 'set in parent'
	
	'name' in child
	//=> true
	
原型联上的 get set 也是会调用的哟

### 初步设计

	xx
	|- src
	    |- api // 开发者使用
	        |- data.js
	        
	        	copy data to scope, sync mechanism
	        
	        |- dom.js
	        |- events.js
	        |- global.js
	        |- lifecycle.js

	    |- instance // 内部，扩展 Vue.prototype
	        |- bindings.js
	        	
	        	setup binding tree with default point
	        	listen ob event to updateBinding
	        
	        |- compile.js
	        |- data.js
	        |- proxy.js
	        	
	        	avoid scope chain, proxy access owner data on self
	        
	        |- scope.js
	        
	        	初始：scope chain, send down upstream change evt
	        	销毁

	    |- vue.js
	    
	    	构造

# [be19525]

> path

	parse('a.b[0].c')
	//=> ["a", "b", "0", "c"]

使用状态机高效的解析出路径 path 中的变量

# [5bff199]

> path and cache


LRU 缓存，数据结构用双向链表，以前在腾讯浏览器面试的时候，用数组（线性表）和索引标记 head/tail，估计是数组 splice 性能不好，面试官不是很满意，目测用双向链表性能好很多吧，学习了

# [14d97d4]

> binding

binding树 树映射 data (对象) 

# [69e9154]

> more util refactor, add build banner

mergeOption 场景:

> 对某些属性有特性的策略去 merge，为了使用父类的一些配置

- extend 创建类的时候

		exports.extend = function (extendOptions) {
		  ...
		  Sub.options = _.mergeOptions(Super.options, extendOptions)

- 构造初始化的时候


		  this.$options = mergeOptions(
		    this.constructor.options,
		    options,
		    this
		  )
		 
# [af194e4]

> expression parser & test

移除了添加 `$parent.` 逻辑，变量前添加 `scope.`

# [c5cb21b]

> test for util

`events` 可以 merge from parent，以前用 Backbone，events 继承要手动折腾，一直很别扭，我喜欢 vue 这样的


# [fb1a149]

> working on directive

binding 的树不使用 .childrend = {} 之类的数据结构，直接用 {} 本身表示一颗 binding 树，而 binding 实例的方法都是 `_` 开头，这样很 hack 方便的建立映射树。

此时，从 path 到 binding 的定位就方便了

	var data = {
		a: {
			b: x
		}
	}

	rootBinding = {
		_addChild: xx,
		...// binding private methods
		a: {// this is binding instance
			_addChild: xx,
			...,
			b: {
				_addChild: xx,
				xxx
			}
			
		}
	}

# [8e710f0]

> add batcher integration to directives

directive 构建机制

		 * @param {String} type
		 * @param {Node} el
		 * @param {Vue} vm
		 * @param {Object} descriptor
		 *                 - {String} arg
		 *                 - {String} expression
		 *                 - {Array<Object>} filters
		 * @constructor
		 */
		
		function Directive (type, el, vm, descriptor)

一个指令可能涉及多个 paths (一个 path 理解成一个变量访问)

首先解析 expression 中的 paths 然后对每个 path 创建 binding，自己（指令）订阅这些 bindings

另外，computed 的依赖侦测则是在构造函数最后一步调用一下 get，大致跟原来的差不多，这个过程会有一个临时成员 this._targetDir，在侦测依赖过程中，所有侦测到的 path 都会创建绑定，并且被 _targetDir 订阅

不足：用于 collectDeps 的 get 事件并没有动态的注册，而是 vm 初始化的时候就注册了这个事件

# [abf6bda]

> optimize expression parser for simple path scenarios

关于 path 几种类型和解析

- a.b.c

这种简单的就直接 Path.compileGetter(['a', 'b', 'c'])

- a['b'].c

有 [x] 这种的，需要状态机解析 Path.parse("a['b'].c") 

- a + b.c + c

带运算的表达式则用到 expression path，

注意，所有路径解析入口都是 expression.parse, 返回的 getter 中 getter.paths 只包含是首级变量 (['a', 'c'])，所有情况都是

为啥需要这部分"首级"变量? 对于注释的这段话也不理解

	 * e.g. in "a && a.b", if `a` is not present at compilation,
	 * the directive will end up with no dependency at all and
	 * never gets updated.

# [d3846f6]

> keep-alive option for v-component

v-component 对应的 vm，在隐藏或者切换的时候不销毁 vm，缓存起来

不足：销毁的时候要把缓存的 vm 也销毁，另外最好给 vm 标记一下是否被使用，免得销毁错了


# [7aa2bab]

> v-repeat diff

很赞的一点，考虑了动画过程可能存在页面但是实际要忽略的 dom 元素，基于 vm.__vue__ 标识一个 dom 是否是一个有效的 vm root el

# [6985f06]

> attached/detached hook logic

		exports.$mount = function (el) {
		  ...
		  this._compile()
		  ...
  		  this.$once('hook:attached', function () {
		    ...
		    this._initDOMHooks()
		  })
		  // 有些动画中导致可能还没插入？
		  if (_.inDoc(this.$el)) {
		    this._callHook('attached')
		  }
		}
	
父元素 attach 之后保证（动画中）的子孩子后续 attach 后也能更新 isAttached 状态
	
		exports._initDOMHooks = function () {
		  var children = this._children
		  this.$on('hook:attached', function () {
		    this._isAttached = true
		    for (var i = 0, l = children.length; i < l; i++) {
		      var child = children[i]
		      if (!child._isAttached && inDoc(child.$el)) {
		        child._callHook('attached')
		      }
		    }
		  })
		}

# [d82785e]

> improve text&attr compile

函数式编程，看的好累，图啥？好多 angular 的影子

# [27874ee]

> greatly simplify observe mechanism

原来通过监听数据改变（其实也是自己 emit 的），再去 binding.notify，现在不要自己 emit，直接 notify 完了，简单好多

# [01e6e9e]

> no longer need an internal emitter

组合？继承？呵呵~

想多了，我觉得就是为了省代码量

# [512d265]

> build 0.11.0-rc

捋一把

    |- / api
        |- child.js
        	
        	创建子 vue，原型链指向自身（父亲）
        
        |- data.js
        	
        	与表达式相关的数据的 get/set/add/delete/eval...
        
        |- dom.js
        
        	dom操作，某些操作保证在动画完成之后
        	
        |- events.js
        	
        	不在依赖 emitter
        
        |- global.js
        	
        	extend, use
        
        |- lifecycle.js
        	
        	$mount, $destroy
        	
    |- / compile
        |- compile.js
        	
        		comiple(el) 返回 function link (vm, el)
        
        |- transclude.js
        
        		对 v-repeat 的每个block 插入开头结尾占位的 comment 节点
        		处理 <content>

    |- directives
        |- attr.js
        |- class.js
        |- cloak.js
        |- component.js
        |- el.js
        |- html.js
        |- if.js
        |- index.js
        |- model
            |- checkbox.js
            |- index.js
            |- radio.js
            |- select.js
            |- text.js
        |- on.js
        |- partial.js
        |- ref.js
        |- repeat.js
        |- show.js
        |- style.js
        |- text.js
        |- transition.js
        |- with.js
    |- filters
        |- array-filters.js
        |- index.js
    |- instance
        |- compile.js
        |- events.js
        	
        	事件机制，检查孩子 attached
        
        |- init.js
        	
        	入口
        
        |- scope.js
        
        	_initData:
        		proxy, new observe and link self (vm)
        		
        	_initComputed:
        		def {get:x,set:x}
        		
        	_initMethods: 
        		proxy methods with content this
        	
        	_initMeta:
        		deps, notify
        	
        	_setData
        		unproxy, unVm old data
        		proxy new key, observe new Data and link vm (self)
        	
        	_digest
        		某一个节点更新，所有子节点都要 digest
        	
        
    |- observer
        |- array.js
        	
        	intercept, $remove/$set
        
        |- index.js
        
        	每个对象/数组创建一个 Observer，对 {} 下每个 key 定义 getter/setter，
        
        |- object.js
        
        	$add: convert, vm proxy and digest or just ob.notify
        	
        	$delete
        	
        	var obj = {name:'peter'} 
			var china = {a:obj}
			var us = {a:obj}
			
			usVm {
			    a: obj
			}
			
			chinaVm {
			    a:obj
			}
			
			
			obj ... ob  |- bindings : [us.a, china.a]
			            |- vms: [usVm, chinaVm]
			
			当 obj.$add sth，需要 vms 中 每个 vm 代理获取、触发 vm 下每个 watcher 更新        	
        
    |- parse
        |- directive.js
        	
        	流解析成 dirs = [{expression:xx,filters:{xx}}, ...]
        
        |- expression.js
        	
        	解析表达式，返回 {get:xx, set:xx}
        
        |- path.js
        	
        	将一个变量表达式解析
        	x.y.z
        	keys = [x,y,z], 
        	make getter keys.get
        
        |- template.js
        	
        	将所有能代表模板的东西转化为 DocumentFragment
        
        |- text.js
        
        	文本中多个绑定变量或者字面字符串解析成 tokens=[{},...]
        
    |- test.js
    |- transition
        |- css.js
        	
        	css 动画
        
        |- index.js
        
        	考虑了动画事件的 dom 操作（如：动画结束后再移除）
        
        |- js.js
        	
        	自定义 enter/leave
        
    |- util
        |- debug.js
        	
        	log 工具
        	
        |- dom.js
        	
        	原生 dom 操作
        
        |- env.js
        	
        	不同执行环境，兼容性
        	
        |- filter.js
        	
        	extract read/write filters, applyFilters
        
        |- index.js
        |- lang.js
        	
        	grammar utils
        
        |- merge-option.js
        	
        	options merge from 
        		(mixin | parent vm | Self Class | Parent Class)
        		
        	1. when Class.extend
        	2. when instance.init
        
                
    |- config.js
    |- directive.js
    	
    	watcher 1 ... n dir
    	watcher.cbs = [dir._update, ..., custom watch func]
    
    |- batcher.js
    	
    	task queue < watcher{id:xx} >
    	buffer 内多次 push 同一个 watcher 会覆盖
    
    |- binding.js
    	
    	{{one exp}} 1 ... 1 binding ... n dir
    	binding.notify ---> each dir.update
    
    |- cache.js
    	
    	双向链表 LRU
    
    |- vue.js
    	
    	extend Vue, Vue.prototpye，def `$data`
    
    |- watcher.js
    
		    .................. < ....................
		    v                                        .
		    .                                        ^
		watcher run ......                           .
		    .            .                           .
		    .            v                           .
		watcher-----|    .                       when change
		            |-- cbs                          .
		            |-- deps...n |-- bindingA --|    .
		                         |-- bindingB --|.. >  
		                         |-- bindingC --|