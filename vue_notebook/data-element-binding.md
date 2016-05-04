# 数据绑定实验

> 源码来自 https://github.com/vuejs/vue/blob/871ed9126639c9128c18bb2f19e6afd42c0c5ad9/explorations%2Fgetset.html

## 总览

- 提供模板

		<div id="test">
			<p>{{msg}}</p>
			<p>{{msg}}</p>
			<p>{{msg}}</p>
			<p>{{what}}</p>
			<p>{{hey}}</p>
		</div>

- 通过js绑定数据

		new Element('test', {msg: 'hello})

- 输出视图

		<div id="test">
			<p><span>hello</span></p>
			<p><span>hello</span></p>
			<p><span>hello</span></p>
			<p><span></span></p>
			<p><span></span></p>
		</div>

## 解剖绑定过程（伪码）

	function Element () {
		//1) 给模板添加一些自定义占位符标志
		
		//2) 根据占位符标找到dom元素
		
		//3) 生成映射 {msg: msg对应的一些dom元素}
		
		//4) 准备数据对象 obj
		
		//5) 使用 Object.defineProperty 定义数据对象的 set, get
		
		//6) 修改数据对象的值
	}

- 1) 给模板添加一些自定义占位符标志

	将
	
		<p>{{msg}}</p>
		
	变成
	
		<p><span data-element-binding="msg"></span></p>
	
	其中 data-element-binding 是自定义的属性（占位符），通过这个属性把 dom 元素和 `{{msg}}` 关联起来

- 2) 根据占位符标找到dom元素
		
		 
		var els = el.querySelectorAll('[data-element-binding="msg"]');

- 3) 生成映射 {msg: msg对应的一些dom元素}	
		
		var bindings = {msg: els};
		/*
		映射对象 => 
		{msg: {
			els: [msg元素1, msgs元素2, ...]，
			value: null //对应数据对象的值
		}}
		*/
- 4) 准备数据对象 obj

		data = {};

- 5) 使用 Object.defineProperty 定义数据对象的 set, get

		for variable in 映射对象
		
			// variable 为 'msg'
			Object.defineProperty(data, variable, {
					set: function (newVal) {
						// 遍历元素
					  bindings[variable].value = 
					  	e.textContent = newVal
					},
					get: function () {
					    return bindings[variable].value
					}
			})

- 6) 修改数据对象的值	


		data.msg = 'hello world';
		// 检测到数据对象属性变化
		// 执行对应 set 方法
		
		// html 被修改 =>
		<div id="test">
			<p><span>hello</span></p>
			<p><span>hello</span></p>
			<p><span>hello</span></p>
			<p><span></span></p>
			<p><span></span></p>
		</div>
	
		// 映射对象更新 =>
		{msg: {
			els: [msg元素1, msgs元素2, ...]，
			value: 'hello world'
		}}
	
		
		
		
		
		
		
		
		
		
		
		
		
		