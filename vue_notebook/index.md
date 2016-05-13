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
		compile node
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
		直接在依赖树的节点写入上层，包括根节点的指令
	}
	
	getSameA = {
		deps: [preABind, aBind],// 依赖关系的表明是夸层级的
		subs: []
	}

