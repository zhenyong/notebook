# 最初的模板解析&数据绑定

> 参考这次提交：[a5e27b1174e9196dcc9dbb0becc487275ea2e84c](https://github.com/vuejs/vue/commit/a5e27b1174e9196dcc9dbb0becc487275ea2e84c)

## 代码解析

	var app = Seed.create({
			    id: 'test',
			    // template
			    scope: {
			        msg: 'hello',
			        hello: 'WHWHWHW',
			        changeMessage: function () {
			            app.scope.msg = 'hola'
			        }
			    }
			})

`id` 指定模板的根容器，`scope` 是需要绑定的数据和方法

Seed 的核心工作就是解析模板，找到自定义的属性，绑定相关数据和方法，下面看下 Seed 的实现（部分伪代码替换源码）

	//...
	
	function Seed (opts) {
		// 1) 【扫描】包含指令（形如 sd-xx） 的元素
		
		// 2) 【处理节点】遍历处理每个包含指令的元素 和 根节点
		
		// 3) 【初始化】
	}

- Seed#【扫描】

	扫描所有包含形如 `sd-xx[-yy]` 的元素, els = [elA, elB, elC, ...]

- Seed#【处理节点】

	- 1) 生成节点的【属性数组】
	
			<p sd-text="msg | capitalize"></p>
	
		=>
			
			attrs = [
				{name:'sd-text',value:'msg | capitalize'}
			]	
	
	
	- 2） 【解析指令】遍历 attrs, 解析 sd 开头的属性
	
		`sd-definition-argument ="key | filters#1 | filters#2"`
		
        	update: Directives#definition 
        	[如果是个对象，则找到对象的 update 方法]
		
		+ 例子1

		对于 `{name:'sd-text',value:'msg | capitalize'}` 
		
		返回指令对象
		
			directive = {
				  el: dom节点,
	            attr: {name:'sd-text',value:'msg | capitalize'},
	            key: 'msg',//绑定变量
	            filters: [ 'capitaliz'],
	            definition: Directives.text方法,
	            argument: null,
	            update: Directives.text方法,
			}
		
		+ 例子2
		
		对于 `sd-on-click="changeMessage | .button"`
		
			directive = {
				...,
				argument:"click",
				filters: ['.button'],
				update: Directives.on对象.update方法
			}

	- 3) 【组成绑定对象】
	
			binding = {
			...,
				msg : {
					directives: [directive#1, directive#2],
					value: //内部实时值(用来更新dom值)
				},
			}
		
	- 4) 【定义 get/set】

	        get: return binding['msg'].value
	        set: function (value) {
	            更新 binding['msg'].value
	            遍历 binding['msg']. directives
	                // 如果 directive.filters 则应用过滤器
	                // 使用过滤器之后的值，调用 directive.update
	            })
	
	
		
		


