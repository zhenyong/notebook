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

### 思考以下重构图啥？

	var filteredValue = value
	if (value && directive.filters) {
		filteredValue = applyFilters(value, directive)
	}
	
=>
	
	var filteredValue = value && directive.filters
	? applyFilters(value, directive)
	: value

我:
 
- 从代码语义看，前者表达方式更像 『你可以...，除非...』，后者像『如果...否则...』
- 从执行效率上看，假如 `value && directive.filters` 为真，那么前者多了一次赋值语句

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