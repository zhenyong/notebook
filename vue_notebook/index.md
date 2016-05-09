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


 
