

#组件开发

[toc]

##组件是否优秀的标准
1. 易用 ( 开箱即用，使用者能够以除了必要参数和配置的方式调用 )
2. 可拓展 （ 业务场景情况个不相同，应该提供尽可能多的配置，方法，事件供使用者拓展 ）
3. 独立(不耦合) （ 组件间应该项目不存在依赖关系，提高复用性 ）

##VUE组件API
###props
`用途`：
1. 组件实例的作用域是孤立的。子组件无法直接引用父组件的数据。所以需要通过指定props显示的申明，并通过父组件将数据传递下来。
2. 对数据做验证和过滤以获取到期望的值。

*数组形式*
```javascript
	Vue.component('ajax-button', {
		props : ['loadingText','url','data']
	});
	//es6
	export default {
		props : ['loadingText','url','data']
	}
	// <ajax-button loading-text="加载中..."></ajax-button>
```
*对象形式*：可对每个参数做验证
```javascript
	Vue.component('ajax-button', {
		props : {
			loadingText: String,
			type : {
				type : 'String',
				coerce: function (val) {
			        return val + '' // 将值转换为字符串
			    }
			}
		}
	});
	//es6
	export default {
		props : {
			loadingText: String,
			url : {
				type: String,
				required : true
			}
		}
	}
	// <ajax-button loading-text="加载中..."></ajax-button>
```
*jQuery实现对比*
```javascript
	$.fn.ajaxButton ＝ function(config){
		//验证
		if(typeof config.loadingText !== 'String'){
			throw new Error('Not String');
			return false;
		}
		if(...){
			...
		}
		//默认参数
		var defaults = {
			loadinigText : '确定',
			...
		}
		var cfg = $.extend(defaults,config);
		//转换
		if(['get','post','put'...].indexOf(cfg.type.toLowerCase()) != -1){
			cfg.type = 'get';
		}
		...
	}
```
**总结**：
1. 类型检查，支持多数据类型(String,Number,Boolean,Function,Object,Array)
2. 可以自定义验证函数
3. 设置默认值 
4. 在数值设置前可以进行转换
5. 双向绑定验证

>组件传递的属性命名是以驼峰命名（camelCase）在html属性中需要转化为中横线

```javascript
	Vue.component('ajax-button', {
		props : ['loadingText']
	});
	// <ajax-button loading-text="加载中..."></ajax-button>
```
###组件通信
####props双向绑定
通过指定 `属性.sync=` 双向绑定，属性的变化会同步到所有组件，这种方式最简单，适合简单的数据通信
>ajax-button中通过通过`loading.sync`将父组件的数据传递给子组件，子组件发送请求时，将数据设置为true，请求完成设置为false，由于是双向绑定，所以父组件可以根据属性在 在加载中和列表之间切换 显示状态。

[demo](http://jsrun.net/Y7KKp?uid=407)


####相互属性引用
1. 父组件通过`this.$children`获取子组件列表
2. 子组件通过`this.$parent`获取父组件
3. 子组件通过`this.$root`获取根组件

根据组件间独立的原则，尽管可以通过这样的方式快速的获取实例通信，但是会让父子组件紧密地耦合，除非是组件本身存在依赖关系。比如选项卡必须项目依赖存在的情况
```javascript
	<tab>
		<tab-item></tab-item>
		<tab-item></tab-item>
	</tab>
```
[demo](http://jsrun.net/i7KKp?uid=407)

####自定义事件
Vue 实例实现了一个自定义事件接口，用于在组件树中通信。这个事件系统独立于原生 DOM 事件，用法也不同。
1. 使用$on( )监听事件
2. 使用$emit( ) 触发事件
3. 使用$dispatch( ) 派发事件，事件沿着父链冒泡
4. 使用$broadcast( ) 广播事件，事件向下传导给所有后代

注
- 和DOM事件不同的是，DOM需要阻止冒泡，在事件处理函数内 `return false` 或者调用 event.stopPropagation(), 自定义事件在遇到事件处理函数时会停止冒泡，需要 `return true`来确定继续冒泡
- 组件初始化时自动调用 `$on( )` 来初始化绑定 events里的事件处理函数，所以可以通过v-on在模板中绑定或者 `$on( )`在钩子函数中版定

[demo](http://jsrun.net/b7KKp?uid=407)


####子组件索引
尽管有 props 和 events，但是有时仍然需要在 JavaScript 中直接访问子组件。通常是一个父组件下有多个相同组件的实例，但是需要获取其中的一个，为此可以使用 v-ref 为子组件指定一个索引 ID。例如：
```html
<template id="ajax-button">
    <button>
      {{text}}
    </button>
</template>

<form id="form">
  <ajax-button v-ref:update text="修改"></ajax-button>
  <ajax-button v-ref:delete text="删除"></ajax-button>
</form>
```
```javascript
	let form = new Vue({
		el : '#form'
	})
	var updateButton = form.$refs.update;
	var deleteButton = form.$refs.delete;
```
[demo](http://jsrun.net/SsKKp?uid=407)

####状态管理
>在大型应用中，状态管理常常变得复杂，因为状态分散在许多组件内。通过以上的方法来通信，项目会变的越来月复杂和不可控。组件需要通信，通常是因为数据使用的相同的数据。 

比如一个页面的有header , footer，login 三个组件 ， header , footer 中要获取登陆状态， login组件负责登陆 

![状态](http://cn.vuejs.org/images/state.png)

[demo](http://jsrun.net/h7KKp?uid=407)


####vuex
>Vuex 是一个专门为 Vue.js 应用所设计的集中式状态管理架构。它借鉴了 Flux 和 Redux 的设计思想，但简化了概念，并且采用了一种为能更好发挥 Vue.js 数据响应机制而专门设计的实现。

1. 后台数据更新或者用户输入操作，触发action的调用；
2. Actions 通过分发 mutations 来修改 store 实例的状态，一个Action可能对应多个mutation
3. Store 实例的状态变化反过来又通过 getters 被组件获知。组件通过getters从store里获取数据
![数据流](http://vuex.vuejs.org/zh-cn/vuex.png)

[文档](http://vuex.vuejs.org/zh-cn/intro.html)

###slot内容分发
以Box组件为例：
[demo](http://jsrun.net/K7KKp?uid=407)

总结
1. 没有指定name的内容分发到匿名的`<slot></slot>` 中；
2. 指定了name的内容分发到同名的`<slot name=""></slot>`;
3. 如果没有分发的内容，使用；`<slot>默认值</slot>`中的内容做为默认值；

###异步组件
>在大型应用中，我们可能需要将应用拆分为小块，只在需要时才从服务器下载。为了让事情更简单，Vue.js 允许将组件定义为一个工厂函数，动态地解析组件的定义。Vue.js 只在组件需要渲染时触发工厂函数，并且把结果缓存起来，用于后面的再次渲染。例如：

```javascript
	Vue.component('async-example', function (resolve, reject) { //Promise
	  setTimeout(function () {
	    resolve({
	      template: '<div>I am async!</div>'
	    })
	  }, 1000)
	})
```
#自定义指令
>除了内置指令(`v-if` `v-for` `v-show`)，Vue.js 也允许注册自定义指令。自定义指令提供一种机制将数据的变化映射为 DOM 行为。

>指令通常被用于拓展已有的DOM的功能，和Angular指令的比，功能要弱很多。 
###属性指令
```javascript
Vue.directive('my-directive', {
  bind: function () {
    // 准备工作
    // 例如，添加事件处理器或只需要运行一次的高耗任务
  },
  update: function (newValue, oldValue) {
    // 值更新时的工作
    // 也会以初始值为参数调用一次
  },
  unbind: function () {
    // 清理工作
    // 例如，删除 bind() 添加的事件监听器
  }
})
```
在注册之后，便可以在 Vue.js 模板中这样用（记着添加前缀 v-）：
```html
	<div v-my-directive="someValue"></div>
```
指令内部的this指向这个指令的实例， this上有一些属性

```html
	<div v-demo:hello.a.b="msg"></div>
	<!--
		name : demo
		expression : msg
		argument : hello  //v-bind:src
		modifiers : {a : true, b: true} // v-on:click.stop.prevent
	-->
```
- el: 指令绑定的元素。
- vm: 拥有该指令的上下文 ViewModel。
- expression: 指令的表达式，不包括参数和过滤器。
- arg: 指令的参数。
- name: 指令的名字，不包含前缀。
- modifiers: 一个对象，包含指令的修饰符。
- descriptor: 一个对象，包含指令的解析结果。
```html
	<!-- 多参数 对象传值 -->
	<div v-demo="{ color: 'white', text: 'hello!' }"></div>
	
```
**注意** 你应当将这些属性视为只读的，不要修改它们。你也可以给指令对象添加自定义属性，但是注意不要覆盖已有的内部属性。 

###元素指令
```javascript
Vue.elementDirective('my-directive', {
  // API 同普通指令
  bind: function () {
    // 操作 this.el...
  }
})
```
```html
	<my-directive></my-directive> <!-- 不建议使用，容易和组件混淆 -->
```

Angular中指令有 A（属性）E（元素）C（Class） M（注释）， Vue中只有AE

###高级功能
1. `params`自定义指令可以接收一个 params 数组，指定一个特性列表，Vue 编译器将自动提取绑定元素的这些特性。
2. `deep` 如果自定义指令用在一个对象上，当对象内部属性变化时要触发 update，则在指令定义对象中指定 deep: true。
3. `toWay`如果指令想向 Vue 实例写回数据，则在指令定义对象中指定 twoWay: true 。该选项允许在指令中使用 this.set(value):
4. `acceptStatement`  传入 acceptStatement:true 可以让自定义指令接受内联语句，就像 v-on 那样：
5. Vue 通过递归遍历 DOM 树来编译模块。
6. priority 可以给指令指定一个优先级。指令优先级高的先处理。

[demo](http://jsrun.net/v7KKp?uid=407)

#过滤器
```javascript
	Vue.filter('reverse', function (value) {
	  return value.split('').reverse().join('')
	})
```
```html
	<!-- 'abc' => 'cba' -->
	<span v-text="message | reverse"></span>
```
很眼熟，参考artTemplate的helper
```javascript
	template.helper('reverse',function (value) {
	  return value.split('').reverse().join('')
	})
```


#插件开发
>插件通常会为 Vue 添加全局功能。插件的范围没有限制——通常是下面几种：

1. 添加全局方法或属性 比如`Vue.extend( )`
2. 添加全局资源 比如 指令 `v-for` `v-touch`，过滤器 `orderBy`  
3. 添加实例方法 比如`vue-router`, `vue-resource` 
```javascript
Plugin.install = function(Vue,options){
	// 1. 添加全局方法或属性
	  Vue.myGlobalMethod = ...
	  // 2. 添加全局资源
	  Vue.directive('my-directive', {})
	  Vue.filter('my-filter',function(value){
	  });
	  Vue.component('com',{})
	  // 3. 添加实例方法
	  Vue.prototype.$myMethod = ...
}
```
调用方式
```javascript
	Vue.use(Plugin)
```
使用方法
1.全局属性方法 `Vue.prop`, `Vue.func( )` 
2.全局资源 `<element my-directive>{{ text | my-filter }}</element>`
2.实力方法，所有的Vue实例( 组件 )中都可以通过 `this.$myMethod` 调用，比如 `this.$http` ,  `this.$router`

**注意**
插件的注册是在全局，所有插件都注册在Vue命名空间下，命名上应该特别注意，避免同名覆盖。

以实例方法为例子：
```javascript
function install(Vue,options){
  var el = document.createElement('span');
  el.className = 'tip';
  for(var key in options){
  	el['style'][key] = options[key];
  }
  el.style.opacity = 0;
  document.body.appendChild(el);
  
  //拓展实例
  Vue.prototype.$tip = function(text,timeout){
  	el.innerHTML = text;
  	el.style.opacity = 1;
    setTimeout(function(){
    	el.style.opacity = 0;
    },timeout || 1000);
  }
}
//注册
Vue.use(install,{
  'fontSize' : '14px',
  'background' : '#f60',
  'color' : '#fff',
  'padding' : '5px 15px'
});

//子组件
Vue.component('child',{
  emplate : '<button @click="showTip">点击提示</button>',
  methods : {
  	showTip : function(){
    	this.$tip('click');
    }
  }
})

// 根组件
new Vue({
	el : 'body',
	ready : function(){
	this.$tip('ready');
	}
})


```
[demo](http://jsrun.net/p7KKp)