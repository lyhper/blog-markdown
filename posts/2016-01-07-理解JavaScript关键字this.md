---
title: 理解JavaScript关键字this
date: 2016-01-07
categories: JavaScript
---

在JavaScript中，this的使用非常灵活，在不同的场景下往往指向不同的内容，但始终满足一个条件，那就是指向调用此函数的对象。以下将列出一些比较常见的情况。

## 1、闭包中的this

```javascript
function test() {
	(function() {
		console.log(this);	
	})(); 
}
test();
```

此时this指向的不是外层函数，而是全局对象window

## 2、在全局作用域下执行函数

```javascript
function test() {
	console.log(this);	
}
test();
```

此时this指向window对象

## 3、对象中的函数

```javascript
var a = {
	test:function(){
		console.log(this);
	}
}
a.test();
```

此时this指向调用它的对象a

## 4、构造函数

```javascript
function Test() {
	this.name = 'xiaoming';
}
var test1 = new Test();
console.log(test1.name);
```

在使用new构造对象时，this会被设置为指向新构造的对象实例

## 5、setTimeout、setInterval

```javascript
var timer = setTimeout(function(){
	console.log(this);
},100);
```

setTimeout、setInterval的执行函数中的this指向window对象

## 6、其他

```javascript
var a = {
	test:function(){
		console.log(this);
	}
}
b = a.test;
b();
```

这里的this会指向window对象而不是a对象，因为引用类型赋值是赋的引用而不是该对象实例，即现在b就是指向test函数，换言之b就是test函数，在全局环境执行b,其中的this当然指向window