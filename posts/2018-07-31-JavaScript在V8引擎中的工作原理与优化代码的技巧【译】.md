---
title: JavaScript在V8引擎中的工作原理与优化代码的技巧【译】
date: 2018-07-31
categories: javasciprt
---

原文地址：[How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

作者：[Alexander Zlatkov](https://blog.sessionstack.com/@zlatkov)

译：[lyhper](https://blog.lyhper.com/)

---


几个星期前，我们开始了一系列旨在深入挖掘JavaScript及其工作原理的工作：我们认为通过了解JavaScript的构建模块以及它们如何共同发挥作用，您将能够编写更好的代码和应用程序。

本系列的第一篇文章重点介绍了引擎，运行时和调用堆栈的概述。 第二篇文章将深入探讨谷歌V8 JavaScript引擎的内部部分。 我们还将提供一些关于如何编写更好的JavaScript代码的建议 —— 我们的SessionStack开发团队在开发产品时遵循的最佳实践。

## 概览

JavaScript引擎是执行JavaScript代码的程序或解释器。 JavaScript引擎可以实现为标准解释器，或即时编译器，它以某种形式将JavaScript编译为字节码。

下面是一个实现JavaScript引擎的热门项目列表：

- V8 —— 由Google开发的开源软件，用C++编写
- Rhino —— 由Mozilla基金会管理，开源，完全用Java开发
- SpiderMonkey —— 第一个支持Netscape Navigator的JavaScript引擎，现在支持Firefox
- JavaScriptCore —— 开源，由Apple为Safari开发
- KJS —— KDE的引擎最初由Harri Porten为KDE项目的Konqueror Web浏览器开发
- Chakra（JScript9）—— Internet Explorer
- Chakra（JavaScript）—— Microsoft Edge
- Nashorn —— 作为OpenJDK的一部分开源，由Oracle Java Languages and Tool Group编写
- JerryScript —— 物联网的轻量级引擎。

## 为什么要创建V8引擎

由谷歌创建的V8引擎是开源的，用C++编写。此引擎在Google Chrome中使用。但是，与其他引擎不同，V8也用于流行的Node.js运行时。

![V8](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/v8.png)

V8最初旨在提高Web浏览器中JavaScript执行的性能。
为了获得速度，V8将JavaScript代码转换为更高效的机器代码，而不是使用解释器。它通过实现JIT（即时）编译器将JavaScript代码编译成机器代码，像许多现代JavaScript引擎一样，如SpiderMonkey或Rhino（Mozilla）。
这里的主要区别是V8不产生字节码或任何中间代码。

## V8曾经有两个编译器

在V8 5.9版本出现之前（今年早些时候发布），该引擎使用了两个编译器：
- full-codegen - 一个简单而快速的编译器，可以生成简单且相对较慢的机器代码。
- Crankshaft - 一种更复杂的（即时）优化编译器，可生成高度优化的代码。

V8引擎还在内部使用多个线程：
- 主线程用于获取代码，编译代码然后执行它
- 一个单独的线程用于编译，因此主线程可以继续执行，而这个线程正在优化代码
- 一个Profiler线程，它将告诉运行时我们花费大量时间运行的方法，以便Crankshaft可以优化它们
- 一些线程来处理垃圾回收扫描

首次执行JavaScript代码时，V8利用full-codegen直接将解析后的JavaScript转换为机器代码而无需任何转换。
这使它可以非常快速地开始执行机器代码。
请注意，V8不使用中间字节码表示，因此无需解释器。

当代码运行一段时间后，profiler线程已经收集了足够的数据来告诉应该优化哪个方法。

接下来，Crankshaft优化从另一个线程开始。
它将JavaScript抽象语法树转换为名为Hydrogen的高级静态单赋值形式（SSA）（译者注：编译器中的概念）表示，并尝试优化Hydrogen图。
大多数优化都是在这个级别完成的。

## 内联

第一个优化是预先内联尽可能多的代码。
内联是用被调用函数的主体来替换调用代码（调用函数的代码行）的过程。
这个简单的步骤使以下优化更有意义。

![内联](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/inline.pnginline)

## 隐藏类

JavaScript是一种基于原型的语言：没有使用克隆过程创建类和对象。JavaScript也是一种动态编程语言，这意味着可以在实例化后轻松地在对象中添加或删除属性。

大多数JavaScript解释器使用类似字典的结构（基于散列函数）来存储对象属性值在内存中的位置。这种结构使得在JavaScript中检索属性的值比在Java或C＃等非动态编程语言中的计算成本更高。在Java中，所有对象属性都是在编译之前由固定对象布局确定的，并且无法在运行时动态添加或删除（好吧，C＃具有动态性类型，这是另一个主题）。因此，属性值（或指向这些属性的指针）可以作为连续buffer存储在存储器中，每个buffer之间具有固定偏移量，可以根据属性类型轻松确定偏移的长度，而这在运行时可以更改属性类型的JavaScript中是不可能的。

由于使用字典在内存中查找对象属性的位置效率非常低，因此V8使用不同的方法：隐藏类。隐藏类的工作方式类似于Java等语言中使用的固定对象布局（类），除非它们是在运行时创建的。现在，让我们看看它们实际上是什么样的：

```javascript
function Point（x，y）{ 
    this.x = x; 
    this.y = y; 
}
var p1 = new Point（1,2）;
```
一旦“new Point（1,2）”调用发生，V8将创建一个名为“C0”的隐藏类。

![隐藏类](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/hidden-class.png)


尚未为Point定义任何属性，因此“C0”为空。

一旦第一个语句“this.x = x”被执行（在“Point”函数内），V8将创建一个名为“C1”的第二个隐藏类，它基于“C0”。“C1”描述了可以找到属性x的存储器中的位置（相对于对象指针）。在这种情况下，“x”存储在偏移 0处，这意味着当将存储器中的点对象视为连续缓冲区时，第一偏移将对应于属性“x”。V8还将使用“类转换”更新“C0”，该类转换指出如果将属性“x”添加到Point对象，则隐藏类应从“C0”切换到“C1”。下面的Point对象的隐藏类现在是“C1”。


![隐藏类](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/hidden-class2.png)

每次将新属性添加到对象时，旧的隐藏类都会更新为指向新隐藏类的转换路径。隐藏类转换非常重要，因为它们允许在以相同方式创建的对象之间共享隐藏类。如果两个对象共享一个隐藏类并且同一属性被添加到它们中，则转换将确保两个对象都接收相同的新隐藏类以及随其附带的所有优化代码。
执行语句“this.y = y”时重复此过程（再次，在Point函数内，在“this.x = x”语句之后）。

创建一个名为“C2”的新隐藏类，将类转换添加到“C1”，声明如果将属性“y”添加到Point对象（已包含属性“x”），则隐藏类应更改为“C2”，Point对象的隐藏类更新为“C2”。

![隐藏类](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/hidden-class3.png)

隐藏类转换取决于属性添加到对象的顺序。看一下下面的代码片段：

```javascript
function Point（x，y）{ 
    this.x = x; 
    this.y = y; 
}
var p1 = new Point（1,2）; 
p1.a = 5; 
p1.b = 6;
var p2 = new Point（3,4）; 
p2.b = 7; 
p2.a = 8;
```

现在，您将假设对于p1和p2，将使用相同的隐藏类和转换。嗯，不是真的。对于“p1”，首先添加属性“a”，然后添加属性“b”。但是，对于“p2”，首先分配“b”，然后是“a”。因此，作为不同转换路径的结果，“p1”和“p2”以不同的隐藏类结束。在这种情况下，以相同的顺序初始化动态属性要好得多，以便可以重用隐藏的类。

## 内联缓存
V8利用另一种技术来优化动态类型语言，称为内联缓存。内联缓存依赖于观察到对相同方法的重复调用往往发生在同一类型的对象上。可以在此处找到对内联缓存的深入解释。

我们将讨论内联缓存的一般概念（如果您没有时间进行上面的深入解释）。

那么它是怎样工作的？V8维护一个在最近的方法调用中作为参数传递的对象类型的缓存，并使用此信息来假设将来作为参数传递的对象类型。如果V8能够对将传递给方法的对象类型做出很好的假设，它可以绕过确定如何访问对象属性的过程，而是使用先前查找到对象的隐藏类的存储信息。

那么隐藏类和内联缓存的概念是如何相关的呢？每当在特定对象上调用方法时，V8引擎必须执行对该对象的隐藏类的查找，以确定访问特定属性的偏移量。在将同一方法成功调用两次到同一个隐藏类之后，V8省略了隐藏类的查找，只是将属性的偏移量添加到对象指针本身。对于该方法的所有未来调用，V8引擎假定隐藏类未更改，并使用先前查找中存储的偏移直接跳转到特定属性的内存地址。这大大提高了执行速度。

内联缓存也是为什么相同类型的对象共享隐藏类非常重要的原因。如果你创建两个相同类型和不同隐藏类的对象（正如我们之前的例子中所做的那样），V8将无法使用内联缓存，因为即使这两个对象属于同一类型，它们对应的隐藏类为其属性分配不同的偏移量。

![内联缓存](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/inline-caching.png)

这两个对象基本相同，但“a”和“b”属性是按不同顺序创建的。

## 编译到机器代码
Hydrogen图优化后，Crankshaft将其降低到称为Lithium的低级别表示。大多数Lithium实现都是特定于体系结构的。寄存器分配发生在此级别。

最后，Lithium被编译成机器代码。然后发生了一些叫做OSR的事情：堆栈替换。在我们开始编译和优化一个明显长期运行的方法之前，我们可能正在运行它。V8不会忘记它只是慢慢执行以重新启动优化版本。相反，它将转换我们拥有的所有上下文（堆栈，寄存器），以便我们可以在执行过程中切换到优化版本。这是一项非常复杂的任务，请记住，在其他优化中，V8最初已经内联了代码。V8并不是唯一能够做到这一点的引擎。

有一种称为去优化的保护措施可以进行相反的转换，它在引擎的假设不再适用的情况下恢复到非优化代码。

## 垃圾收集
对于垃圾收集，V8采用传统的标记和扫描方式来清理。标记阶段应该停止JavaScript执行。为了控制GC成本并使执行更稳定，V8使用增量标记：不是遍历整个堆，尝试标记每个可能的对象，它只是遍历堆的一部分，然后恢复正常执行。下一个GC停止将从上一个堆行走停止的位置继续。这允许在正常执行期间非常短暂的暂停。如前所述，扫描阶段由单独的线程处理。

## Ignition和TurboFan
随着2017年早些时候发布V8 5.9，引入了新的执行管道。这个新的管道在实际的 JavaScript应用程序中实现了更大的性能提升和显着的内存节省。

新的执行管道建立在Ignition，V8的解释器和TurboFan（V8的最新优化编译器）之上。

您可以在[这里](https://v8project.blogspot.com/2017/05/launching-ignition-and-turbofan.html)查看 V8团队关于该主题的博客文章。

自从V8的5.9版本问世以来，V8已经不再使用full-codegen和Crankshaft（自2010年以来为V8提供服务的技术）用于JavaScript执行，因为V8团队一直在努力跟上新的JavaScript语言功能和这些功能需要优化。

这意味着整体V8将具有更简单，更易维护的架构。


![Ignition和TurboFan](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/ignition-turbofan.png)
在Web和Node.js的提升

这些改进只是一个开始。新的Ignition和TurboFan管道为进一步优化铺平了道路，这些优化将在未来几年内提升JavaScript性能并缩小V8在Chrome和Node.js中的占用空间。

最后，这里有一些关于如何编写优化良好的JavaScript的技巧和窍门。您可以从上面的内容中轻松地推导出这些内容，但是，这里是为方便起见的摘要：

## 如何编写优化的JavaScript

1、**对象属性的顺序**：始终以相同的顺序实例化对象属性，以便可以共享隐藏类和随后优化的代码。

2、**动态属性**：在实例化后向对象添加属性将强制隐藏类更改并减慢为先前隐藏类优化的任何方法。因此，在其构造函数中分配所有对象的属性。

3、**方法**：重复执行相同方法的代码将比仅执行一次不同方法的代码运行得更快（由于内联缓存）。

4、**数组**：避免索引不是增量数的稀疏数组。其中不含有每个元素的稀疏数组是哈希表。这种阵列中的元素访问起来更加昂贵。另外，尽量避免预先分配大型数组。尽量动态分配。最后，不要delete数组中的元素。它使数组中的元素稀疏。

5、**标记值**：V8用32位来表示对象和数字。它使用一个位来知道它是一个对象（flag = 1）还是一个称为SMI（SMall Integer）的整数（flag = 0），因为它的31位。然后，如果数值大于31位，则V8将对该数字进行选择，将其变为双精度并创建一个新对象以将数字放入其中。尝试尽可能使用31位带符号的数字，以避免对JS对象进行昂贵的装箱操作。

我们SessionStack尝试在编写高度优化的JavaScript代码时遵循这些最佳实践。原因是，一旦您将SessionStack集成到您的生产Web应用程序中，它就会开始记录所有内容：所有DOM更改，用户交互，JavaScript异常，堆栈跟踪，失败的网络请求和调试消息。 
使用SessionStack，您可以将网络应用中的问题作为视频重播，并查看用户发生的所有事情。所有这一切对您的Web应用程序没有性能影响。
有一个免费的计划，允许您[免费入门](https://app.sessionstack.com/#/signup)。

![建议](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/tips.png)

## 资源
- https://docs.google.com/document/u/1/d/1hOaE7vbwdLLXWj3C8hTnnkpE0qSa2P--dtDvwXXEeD0/pub
- https://github.com/thlorenz/v8-perf
- http://code.google.com/p/v8/wiki/UsingGit
- http://mrale.ph/v8/resources.html
- https://www.youtube.com/watch?v=UJPdhx5zTaw
- https://www.youtube.com/watch?v=hWhMKalEicY