# 深入理解javascript原型和闭包

----------
标签（空格分隔）： 原型和闭包
---

###  ---目录---
[TOC]


[转载 整理至cnblogs](https://www.cnblogs.com/wangfupeng1988/p/3977924.html)
### 说明：

        该教程绕开了javascript的一些基本的语法知识，直接讲解javascript中最难理解的两个部分，也是和其他主流面向对象语言区别最大的两个部分——原型和闭包，当然，肯定少不了原型链和作用域链。帮你揭开javascript最神秘的面纱。
        
        为什么要偏偏要讲这两个知识点？
        
        这是我在这么多年学习javascript的经历中，认为最难理解、最常犯错的地方，学习这两个知识点，会让你对javascript有更深层次的理解，至少理解了原型和作用域，就不能再算是javascript菜鸟了。另外，这两方面也是javascript与其他语言不同的地方，学习这样的设计，有助于你开阔眼界，帮助你了解编程语言的设计思路。毕竟，你不能只把眼睛盯在一门语言上。
        
        闲话不多讲，相信奔着这个话题来的朋友，也知道javascript原型和作用域的重要性。
        
        最后说明：被系列教程我不是照搬的其他图书或者网络资料，而是全凭着我对知识的理解而一步一步写的。思路也是我一边写着一边想的。有什么不对的地方，欢迎指正。

### 1. 深入理解javascript原型和闭包（1）——一切都是对象

    “一切都是对象”这句话的重点在于如何去理解“对象”这个概念。当然，也不是所有的都是对象，值类型就不是对象。
首先咱们还是先看看javascript中一个常用的运算符——typeof。typeof应该算是咱们的老朋友，还有谁没用过它？
typeof函数输出的一共有几种类型，在此列出：
```
function show(x) {
    console.log(typeof x);    // undefined
    console.log(typeof 10);   // number
    console.log(typeof 'abc'); // string
    console.log(typeof true);  // boolean
    
    console.log(typeof function () {});  //function

    console.log(typeof [1, 'a', true]);  //object
    console.log(typeof { a: 10, b: 20 });  //object
    console.log(typeof null);  //object
    console.log(typeof new Number(10));  //object
        }
    show();
```
以上代码列出了typeof输出的集中类型标识，其中上面的四种（undefined, number, string, boolean）属于简单的值类型，不是对象。剩下的几种情况——函数、数组、对象、null、new Number(10)都是对象。他们都是引用类型。

    判断一个变量是不是对象非常简单。值类型的类型判断用typeof，引用类型的类型判断用instanceof。
```
var fn = function () { };
console.log(fn instanceof Object);  // true
```
好了，上面说了半天对象，各位可能也经常在工作中应对对象，在生活中还得应对活生生的对象。有些个心理不正常或者爱开玩笑的单身人士，还对于系统提示的“找不到对象”耿耿于怀。那么在javascript中的对象，到底该如何定义呢？

    对象——若干属性的集合。
java或者C#中的对象都是new一个class出来的，而且里面有字段、属性、方法，规定的非常严格。但是javascript就比较随意了——数组是对象，函数是对象，对象还是对象。对象里面的一切都是属性，只有属性，没有方法。那么这样方法如何表示呢？——方法也是一种属性。因为它的属性表示为键值对的形式。

而且，更加好玩的事，javascript中的对象可以任意的扩展属性，没有class的约束。这个大家应该都知道，就不再强调了。
先说个最常见的例子：
![图1：][1]
  [1]: https://images0.cnblogs.com/blog/138012/201409/172012532064920.png
以上代码中，obj是一个自定义的对象，其中a、b、c就是它的属性，而且在c的属性值还是一个对象，它又有name、year两个属性。

这个可能比较好理解，那么函数和数组也可以这样定义属性吗？——当然不行，但是它可以用另一种形式，总之函数/数组之流，只要是对象，它就是属性的集合。
以函数为例子：
```
var fn = function () {
    alert(100);
};
fn.a = 10;
fn.b = function () {
    alert(123);
};
fn.c = {
    name: "王福朋",
    year: 1988
};
```
上段代码中，函数就作为对象被赋值了a、b、c三个属性——很明显，这就是属性的集合吗。

你问：这个有用吗？
回答：可以看看jQuery源码！

在jQuery源码中，“jQuery”或者“$”，这个变量其实是一个函数，不信你可以叫咱们的老朋友typeof验证一下。
```
console.log(typeof $);  // function
console.log($.trim(" ABC "));
```
验明正身！的确是个函数。那么咱们常用的 $.trim() 也是个函数，经常用，就不用验了吧！

很明显，这就是在$或者jQuery函数上加了一个trim属性，属性值是函数，作用是截取前后空格。

javascript与java/C#相比，首先最需要解释的就是弱类型，因为弱类型是最基本的用法，而且最常用，就不打算做一节来讲。

其次要解释的就是本文的内容——一切（引用类型）都是对象，对象是属性的集合。最需要了解的就是对象的概念，和java/C#完全不一样。所以，切记切记！

最后，有个疑问。在typeof的输出类型中，function和object都是对象，为何却要输出两种答案呢？都叫做object不行吗？——当然不行。

具体原因，且听下回分解！

### 2. 深入理解javascript原型和闭包（2）——函数和对象的关系

上文（理解javascript原型和作用域系列（1）——一切都是对象）已经提到，函数就是对象的一种，因为通过instanceof函数可以判断。
```
var fn = function () { };
console.log(fn instanceof Object);  // true
```
对！函数是一种对象，但是函数却不像数组一样——你可以说数组是对象的一种，因为数组就像是对象的一个子集一样。但是函数与对象之间，却不仅仅是一种包含和被包含的关系，函数和对象之间的关系比较复杂，甚至有一点鸡生蛋蛋生鸡的逻辑，咱们这一节就缕一缕。

还是先看一个小例子吧。
```
function Fn() {
    this.name = '王福朋';
    this.year = 1988;
    }
var fn1 = new Fn();
```
上面的这个例子很简单，它能说明：对象可以通过函数来创建。对！也只能说明这一点。

但是我要说——对象都是通过函数创建的——有些人可能反驳：不对！因为：
```
var obj = { a: 10, b: 20 };
var arr = [5, 'x', true];
```
但是不好意思，这个——真的——是一种——“快捷方式”，在编程语言中，一般叫做“语法糖”。

做“语法糖”做的最好的可谓是微软大哥，它把他们家C#那小子弄的不男不女从的，本想图个人见人爱，谁承想还得到处跟人解释——其实它是个男孩！

话归正传——其实以上代码的本质是：
```
//var obj = { a: 10, b: 20 };
//var arr = [5, 'x', true];

var obj = new Object();
obj.a = 10;
obj.b = 20;

var arr = new Array();
arr[0] = 5;
arr[1] = 'x';
arr[2] = true;
```
而其中的 Object 和 Array 都是函数：
```
console.log(typeof (Object));  // function
console.log(typeof (Array));  // function
```
所以，可以很负责任的说——对象都是通过函数来创建的。

现在是不是糊涂了——对象是函数创建的，而函数却又是一种对象——天哪！函数和对象到底是什么关系啊？

别着急！揭开这个谜底，还得先去了解一下另一位老朋友——prototype原型。

### 3. 深入理解javascript原型和闭包（3）——prototype原型 

继typeof之后的另一位老朋友！

prototype也是我们的老朋友，即使不了解的人，也应该都听过它的大名。如果它还是您的新朋友，我估计您也是javascript的新朋友。

在咱们的第一节（深入理解javascript原型和闭包（1）——一切都是对象）中说道，函数也是一种对象。他也是属性的集合，你也可以对函数进行自定义属性。

不用等咱们去试验，javascript自己就先做了表率，人家就默认的给函数一个属性——prototype。对，每个函数都有一个属性叫做prototype。

这个prototype的属性值是一个对象（属性的集合，再次强调！），默认的只有一个叫做constructor的属性，指向这个函数本身。

![图2：][1]
  [1]: https://images0.cnblogs.com/blog/138012/201409/172121182841896.png

如上图，SuperType是是一个函数，右侧的方框就是它的原型。

原型既然作为对象，属性的集合，不可能就只弄个constructor来玩玩，肯定可以自定义的增加许多属性。例如这位Object大哥，人家的prototype里面，就有好几个其他属性。

![图3：][1]
  [1]: https://images0.cnblogs.com/blog/138012/201409/172130097842386.png

咦，有些方法怎么似曾相似？

对！别着急，之后会让你知道他们为何似曾相识。

接着往下说，你也可以在自己自定义的方法的prototype中新增自己的属性。
```
function Fn() { }
    Fn.prototype.name = '王福朋';
    Fn.prototype.getYear = function () {
        return 1988;
    };
```
看到没有，这样就变成了

![图4：][1]
  [1]: https://images0.cnblogs.com/blog/138012/201409/172138591437263.png

没问题！

但是，这样做有何用呢？ —— 解决这个问题，咱们还是先说说jQuery吧。
```
var $div = $('div');
$div.attr('myName', '王福朋');
```

以上代码中，$('div')返回的是一个对象，对象——被函数创建的。假设创建这一对象的函数是 myjQuery。它其实是这样实现的。
```
myjQuery.prototype.attr = function () {
    //……
};
$('div') = new myjQuery();
```
不知道大家有没有看明白。

如果用咱们自己的代码来演示，就是这样
```
function Fn() { }
    Fn.prototype.name = '王福朋';
    Fn.prototype.getYear = function () {
        return 1988;
    };
var fn = new Fn();
console.log(fn.name);
console.log(fn.getYear());
```
即，Fn是一个函数，fn对象是从Fn函数new出来的，这样fn对象就可以调用Fn.prototype中的属性。

因为每个对象都有一个隐藏的属性——“__proto__”，这个属性引用了创建这个对象的函数的prototype。即：fn.__proto__ === Fn.prototype

这里的"__proto__"成为“隐式原型”，下回继续分解。












