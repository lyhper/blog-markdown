---
title: jquery之deferred/promise详解
categories: JavaScript
date: 2016-05-13
---
jquery从1.5版本开始引入deferred对象，旨在提供一套回调函数解决方案。

## what's deferred/promise?
deferred（延迟）是由$.Deferred()创建的一个对象，它可用于以链式调用的语法将多个回调函数注册到一个回调函数队列里。deferred对象有三种状态——进行时、已成功、已失败，处于已成功状态时则调用.done()方法指定的函数，处于已失败状态时则调用.fail()方法指定的函数，处于进行时状态则调用.progress()方法指定的函数。deferred对象包含resolve()和reject()两个方法，分别将自己的状态设置为已成功和已失败，从而判定接下来调用哪个函数。promise对象相当于deferred对象的精简版，包含.done()，.fail()等方法，移除了.resolve(),.reject()等改变自身状态的方法，从而防止执行状态被外部修改。

## how to use deferred/promise?

### 1、ajax操作
jquery中ajax操作会返回一个promise对象，如var promise = $.ajax('/')，因此ajax操作后可以直接使用deferred对象的所有方法（除了resolve和reject等改变自身状态的方法）。举个例子：
```javascript
$.ajax('/').done(function(data){
	// 一些操作
}).fail(function(data){
	// 一些操作
})
```

### 2、普通操作
为普通操作添加回调函数需要调用$.Deferred()方法自主创建deferred对象，代码如下：
```javascript
function wait(){
	var drd = $.Deferred();
	setTimeout(function(){
		console.log('操作已完成');
		drd.resolve();
	},3000);
	// 返回promise对象而不是deferred对象，防止意外修改deferred对象的状态
	return drd.promise();
}
wait().done(function(){
	console.log('后续操作');
})
```

## what're advantages of deferred/promise?

### 1、允许添加多个回调函数
```javascript
$.ajax('/').done(function(){
	// 第一个成功回调
}).done(function(){
	// 第二个成功回调
})
```
注册的多个回调函数将依照注册顺序执行，但并不是后一个操作必须等到前一个操作执行完毕再执行，而是前一个操作执行后（无论是否执行完毕）将立即执行下一个回调函数。如果后一个回调函数依赖于前一个回调函数的结果，则可以使用.then()方法，例如：
```javascript
$.ajax('/').then(function(){
	setTimeout(function(){
		console.log('1');
	},3000);
}).then(function(){
	console.log('2')
})
```
这里的执行结果会是先打印2，再打印1。因为第一个.then()函数传入的方法并未返回一个新的deferred对象，因此会直接返回第一个.then()函数默认的deferred对象，而此对象状态在执行第一个.then()方法时已经完成了。因此会直接执行第二个回调函数。
```javascript
$.ajax('/').then(function(){
	var drd = $.Deferred();
	setTimeout(function(){
		console.log('1');
		drd.resolve();
	},3000);
	return drd.promise();
}).then(function(){
	console.log('2')
})
```
这里的执行结果会是符合预期的，即先打印1，再打印2。这里第二个回调函数会等待第一个回调函数执行完毕后再执行，因为第一个.then()函数传入的方法返回了一个自己创建的deferred对象，从而覆盖了默认deferred对象的执行状态。

### 2、为多个操作指定回调函数
使用$.when()可以为多个操作指定回调函数，例如：
```javascript
$.when($.ajax('/test1'),$.ajax('/test2')).done(function(xhr1,xhr2){
	// 一些操作
})
```
当所有操作均执行成功则调用.done()方法指定的回调函数,且回调函数返回的参数为xhr对象而非数据。只有当$.when()只传入了一个deferred对象时回调函数返回的参数才是数据。

### 3、为所有普通操作指定回调函数
deferred/promise使得不仅可以为ajax操作添加回调函数，还可以为其他普通操作添加回调函数，示例见上。