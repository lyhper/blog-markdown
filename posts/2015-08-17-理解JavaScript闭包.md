---
title: 理解JavaScript闭包
date: 2015-08-17
---

闭包就是一个函数，而这个函数可以访问其他函数作用域内的变量。常见的创建闭包的方式，就是在一个函数的内部创建另一个函数。如下所示：

```javascript
function function1(){
    var name="function1";
    return function2(){
        alert(name);
    }
}
var f=function1();
f();//弹窗显示function1
```

这样就创建了一个简单的闭包，之所以返回的函数function2还可以访问name属性，是因为function2中的作用域链还包括function1的作用域。因此内存中仍存在这个活动对象，一旦返回的函数function2被销毁，这个活动对象也会被销毁。可通过解除对function2的引用来释放内存（f=null）。
闭包包含的是整个变量对象，而不是某个具体的变量。由于闭包会携带包含它的函数的作用域，因此过度使用闭包会导致内存占用过高。
值得注意的是，闭包只能取得包含函数中变量的最后一个值。举个例子：

```javascript
function f1(){
    var  result=[];
    for(var i =0;i<10;i++){
    result[i]=function(){
                return i;
            }
    }
    return result;
}
var funcs=f1();
for(var i=0;i<10;i++){
    console.log(funcs[i]());
}
```

以上代码会打印10次i，而且每个函数都返回的是10。因为每个函数引用的都是同一个变量i，即f1函数中的i，而i在f1函数中经过迭代，最终变为了10。
我们可以通过创建另一个匿名函数的方式让闭包从1到10挨个返回，如下所示：

```javascript
function f1(){
    var  result=[];
    for(var i =0;i<10;i++){
        //此处添加一个匿名函数
        result[i]=function(num){
              return function(){
                    return num;
              }
        }(i);
    }
    return result;
}
var funcs=f1();
for(var i=0;i<10;i++){
    console.log(funcs[i]());
}
```

这样处理就能使结果符合预期，从1到10打印。因为数组中的每个函数的作用域链包含的都是不同的匿名函数作用域，且其中的num变量也不同。
