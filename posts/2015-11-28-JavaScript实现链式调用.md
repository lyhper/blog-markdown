---
layout: post
title: JavaScript实现链式调用
date: 2015-11-28
categories: JavaScript
---

用过jQuery的话，对链式调用肯定不会陌生，像这样:$("#test").mouseenter().mouseleave(),这就是链式调用，这样写的话可以节省代码量，而不用像这样写:$("#test").mouseenter()；$("#test").mouseleave()。在js里要是这样写的话，肯定是会报错的，要想实现这样的效果需要进行一些特殊的处理。

## 在方法内返回对象自身

```javascript
var a = {
	f1:function(){
		console.log("I am f1");
		return this;
	},
	f2:function(){
		console.log("I am f2");
		return this;
	},
	f3:function(){
		console.log("I am f3");
		return this;
	}
}
a.f1().f2().f3();
```

这样就很轻松地实现了链式调用

## 返回函数自身

```javascript
function chain(obj){
	//将数组的slice方法赋值给slice变量，方便后边调用
	var slice = Array.prototype.slice; 
	
	return function(){
		//如果未传参数，则直接返回对象
		if(arguments.length === 0){
			return obj;
		}
		//这句可能不太好理解，下边详细解释
		obj[arguments[0]].apply(obj,slice.call(arguments,1));
		//返回函数本身
		return arguments.callee;
	}
}
var a = {
	f1:function(i){
		console.log("I am f1,the variable is "+i);
	},
	f2:function(i){
		console.log("I am f2,the variable is "+i);
	},
	f3:function(i){
		console.log("I am f3,the variable is "+i);
	}
}
chain(a)("f1",1)("f2",2)("f3",3);
```

采用这种方式实现的链式调用，其调用方式会与jQuery那种"点"+“函数”的方式有些不同，这种方式是直接跟一对括号，括号里边第一个参数是obj的函数名，后边的参数则是需要传递给obj里边函数的参数。

解释一下这句obj[arguments[0]].apply(obj, slice.call(arguments,1))，首先看slice.call(arguments,1)，这句是将arguments这个类数组对象转化为数组，且是从类数组对象中第二个值开始，因为第一个值是传过来的obj里的函数名；再看obj[arguments[0]]，这句就是取得obj内指定的函数;这样一来就清晰了，这句话实际上意思就是执行那个obj里的函数，并把参数传进去。