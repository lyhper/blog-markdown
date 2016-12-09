JavaScript主要通过八种方式创建对象，包括：
- 创建Object的实例
- 工厂模式
- 构造函数模式
- 原型模式
- 构造函数和原型模式相结合
- 动态原型模式
- 寄生构造函数模式
- 稳妥构造函数模式

---

## 创建Object的实例
最简单的方式是创建一个Object的实例，然后再为其添加属性和方法，如下所示：
```javascript
var person=new Object();
//添加属性
person.name="xiaoming";
//添加方法
person.sayName=function(){
  alert(this.name);
}
person.sayName();//xiaoming
```
但使用这种方法有一个缺点，就是会产生大量冗余代码。

## 工厂模式
工厂模式就是用函数来封装创建对象的细节，如下所示：
```javascript
function person(name){
  var o =new Object();
  o.name=name;
  o.sayName=function(){
    alert(this.name); 
  }
  return o;
}
var person1=person("xiaoming");
person1.sayName();//xiaoming
```
使用这种方法也有缺点，那就是无法确定这个对象属于哪个类别，即对象识别问题

## 构造函数模式
构造函数可以用来创造特定类型的对象，比如Object这种原生构造函数。构造函数一般会将其首字母大写以作区分。构造函数其实也是函数，只不过专门被用于创建对象。代码如下所示：
```javascript
function Person(name){
  this.name=name;
  this.sayName=function(){
    alert(this.name);
  }
}
var person1=new Person("xiaoming");
person1.sayName();//xiaoming
```
用new操作符执行这个函数会经历如下步骤：
1、创建对象；
2、将构造函数作用域赋给新对象，使this指向新对象；
3、执行构造函数中的代码；
4、返回新对象；

当然，构造函数模式也不是没有缺点，实际上用这种方式创建的不同对象其方法是不同的function的实例，即不同的方法，只不过其名字相同。

## 原型模式
每一个函数都有一个prototype属性，即原型属性，它是一个对象，包含了所有同一类型的实例所共享的属性和方法。代码如下：
```javascript
function Person(){};
Person.prototype.name="xiaoming";
Person.prototype.sayName=function(){
  alert(this.name);
}
var person1=new Person();
person1.sayName();//xiaoming
```

通过原型模式创建的同一类型的对象将共享这些属性和方法，而这也带来了新的问题——事实上，每个对象的属性并不应该被共享。

## 构造函数与原型模式相结合
这种方法就是将对象的属性通过构造函数创建，而将对象的方法通过原型的方式创建。代码如下：
```javascript
function Person(name){
  this.name=name;
}
Person.prototype.sayName=function(){
  alert(this.name);
}
var person1=new Person("xiaoming");
person1.sayName();//xiaoming
```

这是最常见的创建类型的方法，通过这种方法创建的对象将共享方法，而不共享属性

## 动态原型模式
观察上一种模式，你会发现构造函数与原型是分离的，而动态原型将所有的信息都封装在了一起，并且同时使用了构造函数和原型的优点。代码如下：
```javascript
function Person(name){
  this.name=name;
  if(typeof this.sayName!="function"){
    Person.prototype.sayName=function(){
      alert(this.name);
    }
  }
}
var person1=new Person("xiaoming");
person1.sayName();//xiaoming
```

## 寄生构造函数模式
代码如下：
```javascript
function Person(name){
  var o = new Object();
  o.name=name;
  o.sayName=function(){
    alert(this.name);
  }
  return o;
}
var person=new Person("xiaoming");
person.sayName();//xiaoming
```

其实这个方法除了使用new关键字和函数名首字母大写外，与工厂模式完全相同。构造函数默认会返回新对象的实例，而此代码中return o将覆盖构造函数返回的值

## 稳妥构造函数模式
要了解稳妥构造函数，首先得知道稳妥对象，即没有公共属性，其方法也不引用this对象的对象。稳妥对象适合一些安全的环境，这种环境往往需要防止数据被其他应用程序改动。代码如下：
```javascript
function Person(name){
  var o = new Object();
  o.sayName=function(){
    alert(name); 
  }
  return o;
}
var person=Person("xiaoming");
person.sayName();//xiaoming
```

person保存的便是稳妥对象，这种方法创建的对象将只能通过sayName方法访问name属性
