JavaScript主要通过以下6种方式来实现对象的继承，包括:
- 原型链
- 构造函数
- 组合继承
- 原型式继承
- 寄生式继承
- 寄生组合式继承

---

## 原型链
此方法主要是通过将父类的实例赋值给子类的原型，从而将父类的属性与方法存于子类的原型中，实现继承。代码如下：
```javascript
function Food(){
  this.isEatable=true;
}
Food.prototype.getIsEatable=function(){
  alert(this.isEatable);
}
function Fruit(){
  this.hasWater=true;
}

//实现继承
Fruit.prototype=new Food();

Fruit.prototype.getHasWater=function(){
  alert(this.hasWater);
}
var fruit1=new Fruit();
fruit1.getIsEatable();//true
console.log(fruit1 instanceof Food);//true
```

当要查找某个属性或方法时，会沿着原型链向上查找，直到找到为止。即首先查找实例，然后查找子类的原型，最后查找父类的原型。

## 构造函数

这种方法的主要思想是在子类的作用域中调用父类的构造函数实现继承。代码如下：
```javascript
function Food(){
  this.isEatable=true;
}
function Fruit(){
  this.hasWater=true;
  //实现继承
  Food.call(this);
}
var fruit1=new Fruit();
console.log(fruit1.isEatable);//true
```

构造函数的方法实现继承也会存在一个问题，那就是方法的复用。因此，实际应用中很少会单独使用构造函数的方法。

## 组合继承

组合继承实际上就是将以上两种方式相结合的一种方法，代码如下：
```javascript
function Food(){
  this.isEatable=true;
}
Food.prototype.getIsEatable=function(){
  alert(this.isEatable);
}
function Fruit(){
  this.hasWater=true;
  //构造函数继承属性
  Food.call(this);
}
//原型链方式继承属性和方法
Fruit.prototype=new Food();

Fruit.prototype.getHasWater=function(){
  alert(this.hasWater);
}
var fruit1=new Fruit();
fruit1.getIsEatable();//true
```

这种方法结合了构造函数方式和原型链方式的优点，同时避免了单独使用任一方式的缺点，它也是最常用的实现继承的方法。

## 原型式继承

原型式继承的主要思想是基于已有的对象通过原型创建新对象，代码如下：
```javascript
var person={
  name:“xiaoming",
  friends:["xiaohong","xiaogang"]
};
function createPerson(o){
  function F(){};
  F.prototype=o;
  return new F();
}
var person1=createPerson(person);
person1.name="xiaoli";
console.log(person1.name);//xiaoli
person1.friends.push("xiaozhang");
console.log(person1.friends);//xiaohong,xiaogang,xiaozhang

var person2=createPerson(person);
person2.friends.push("xiaoling");
console.log(person2.friends);//xiaohong,xiaogang,xiaozhang,xiaoling
```

因为createPerson里将person的属性赋给了F的原型，因此这些属性将被所有创建的对象所共享

## 寄生式继承

寄生式继承相当于是对原型式继承的一种增强，添加了每个对象自己的方法，代码如下：
```javascript
function object(o){
  function F(){};
  F.prototype=o;
  return new F();
}
function createPerson(original){
  var temp=object(original);
  temp.sayName=function(){
    alert(this.name);
  }
  return temp;
}
var person={
  name:"xiaoming",
  friends:["xiaozhang","xiaowang","xiaoli"]
};
var person1=createPerson(person);
person1.sayName();//xiaoming
console.log(person1.friends);//xiaozhang,xiaowang,xiaoli
```

这种方法也存在一个问题，那就是方法的复用问题，每个对象的方法同名不同质

## 寄生组合式继承

寄生组合式继承的主要思想就是利用构造函数和复制原型对象的方式实现继承，代码如下：
```javascript
function object(o){
  function F(){};
  F.prototype=o;
  return new F();
}
function inheritPrototype(superType,subType){
  var prototype=object(superType.prototype);
  prototype.constructor=subType;//将原型对象的构造函数属性指向子类
  subType.prototype=prototype;
}
function SuperType(){
  this.name="xiaoming";
}
SuperType.prototype.sayName=function(){
  alert(this.name);
}
function SubType(){
  Super.call(this);//调用超类构造函数继承属性
  this.age=21;
}
inheritPrototype(SuperType,SubType);//将超类原型复制到子类原型
Subtype.prototype.sayAge=function(){
  alert(this.age);
}
var person1=new SubType();
person1.sayName();//xiaoming
```

利用这种方式实现继承只调用了一次超类构造函数，从而避免了在原型上创建不必要的属性









