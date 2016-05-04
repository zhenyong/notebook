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

