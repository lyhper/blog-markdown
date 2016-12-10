---
title: 理解JavaScript定时器
date: 2016-01-28
categories: JavaScript
---

首先需要明白的是，javascript是运行于单线程的环境中，没有任何代码是立即执行的，而是等到进程空闲后尽快执行。因此，当设定一个200ms的定时器时，并不代表200ms后立刻执行这段代码，而是表示200ms后将定时器的代码添加到队列里。当队列空闲时，则执行此段代码。

## 使用setInterval需谨慎

```javascript
setInterval(function(){
	test();
},200);
```

多个定时器代码执行的间隔可能会比200ms小，如果此时test函数执行时间过长，在200ms之后仍在执行，则定时器会将test函数代码继续添加到队列中。当第一次test执行完毕后立即执行第2次test函数，这意味这两次代码的执行间隔不存在了。

因此，我们尽量使用递归调用setTimeout的方式来模拟setInterval，如下所示：

```javascript
function interval (){
	setTimeout(function(){
		test();
		interval();
	},200);
}
```

尽量不要使用arguments.callee，因为在严格模式下，argeuments.callee会被禁用。

## 切换标签后的定时器间隔

当切换标签使当前标签页处于后台时，现代浏览器可能是出于节省资源的目的，会将定时器间隔变大。这意味着使用定时器制作动画的时候尤其要注意这一点，因为这可能导致动画在前台时一切正常，而当切换标签再切换回来时动画就错乱了。因此，在利用定时器制作动画时应当采用setTimeout模拟的方式，这样可以保证前一个定时器代码执行完之后再将新的定时器代码插入队列。