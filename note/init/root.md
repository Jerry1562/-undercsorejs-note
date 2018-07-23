## `underscore`的实现原理
`underscore`是一个工具函数集，那么该如何去组织这些函数呢？这些如果按照我的思路，大概可以写成下面的样子：
```javascript
(function(){
	var root = this;

	var _ = {};

	root._ = _;

	//在这里添加方法
	_.sum = function(arr,func){
		for(let i=0;i<arr.length;i++){
			func(arr[i]);
		}
	}
})();

_.sum([1,2],console.log);  //=>1  =>2
```
首先我们要创建一个对象`_`，用于保存所有的方法，然后把该对象挂载到全局环境中。  
为什么不使用`window._ = _`呢，是考虑到框架要运行在不同的全局环境下，不仅仅是window，还有global等。 

>### Tips
>这里使用了一个JavaScript中常见的技术，通过创建一个匿名函数并立即调用它，实现块级作用域的功能。在一般情况下，我们应该尽量避免向全局环境中添加太多的变量，特别是由多个开发人员共同参与一个项目时，特别容易出现命名冲突的问题。通过创建私有作用域，开发人员可以自由地使用自己的变量而不担心污染全局环境。

## `root`变量的获取
如果我们要把`_`挂载到全局环境，首先就要获取全局对象。在上面的代码中，我们使用`var root = this;`获取全局对象，但是，使用`this`会有两点限制：
+ 严格模式下`this`指向全局对象会返回`undefined`。
+ 在ES6的模块中`this`指向顶层环境会返回`undefined`。

代码运行在这两种情境下会报错，为了避免这个问题，我们只能放弃使用`this`，手动匹配全局对象。思路是先进行环境检测，然后把`_`对象挂载到对应的全局对象中。改进后的代码如下：
```javascript
var root = typeof global == 'object' && global.global === global && global ||
	   typeof window == 'object' && window.window === window && window ||
```
在这里，要解析一下`global.global === global`
