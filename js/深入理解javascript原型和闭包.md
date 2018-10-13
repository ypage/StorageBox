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

![图1][1]

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

![图2：][2]

如上图，SuperType是是一个函数，右侧的方框就是它的原型。

原型既然作为对象，属性的集合，不可能就只弄个constructor来玩玩，肯定可以自定义的增加许多属性。例如这位Object大哥，人家的prototype里面，就有好几个其他属性。

![图3：][3]

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

![图4：][4]

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

### 4. 深入理解javascript原型和闭包（4）——隐式原型

注意：本文不是javascript基础教程，如果你没有接触过原型的基本知识，应该先去了解一下，推荐看《javascript高级程序设计（第三版）》第6章：面向对象的程序设计。

![图5：][5]

![图6：][6]

上面截图看来，obj.__proto__和Object.prototype的属性一样！这么巧！

答案就是一样。

obj这个对象本质上是被Object函数创建的，因此obj.__proto__=== Object.prototype。我们可以用一个图来表示。

![图7：][7]

即，每个对象都有一个__proto__属性，指向创建该对象的函数的prototype。

那么上图中的“Object prototype”也是一个对象，它的__proto__指向哪里？

好问题！

在说明“Object prototype”之前，先说一下自定义函数的prototype。自定义函数的prototype本质上就是和 var obj = {} 是一样的，都是被Object创建，所以它的__proto__指向的就是Object.prototype。

但是Object.prototype确实一个特例——它的__proto__指向的是null，切记切记！

![图8：][8]
 
还有——函数也是一种对象，函数也有__proto__吗？

又一个好问题！——当然有。

函数也不是从石头缝里蹦出来的，函数也是被创建出来的。谁创建了函数呢？——Function——注意这个大写的“F”。

且看如下代码。
 
![图9：][9]

以上代码中，第一种方式是比较传统的函数创建方式，第二种是用new Functoin创建。

首先根本不推荐用第二种方式。

这里只是向大家演示，函数是被Function创建的。

好了，根据上面说的一句话——对象的__proto__指向的是创建它的函数的prototype，就会出现：Object.__proto__ === Function.prototype。用一个图来表示。

![图10：][10]

上图中，很明显的标出了：自定义函数Foo.__proto__指向Function.prototype，Object.__proto__指向Function.prototype，唉，怎么还有一个……Function.__proto__指向Function.prototype？这不成了循环引用了？

对！是一个环形结构。

其实稍微想一下就明白了。Function也是一个函数，函数是一种对象，也有__proto__属性。既然是函数，那么它一定是被Function创建。所以——Function是被自身创建的。所以它的__proto__指向了自身的Prototype。

篇幅不少了，估计也都看烦了。快结束了。

最后一个问题：Function.prototype指向的对象，它的__proto__是不是也指向Object.prototype？

答案是肯定的。因为Function.prototype指向的对象也是一个普通的被Object创建的对象，所以也遵循基本的规则。

![图11：][11]

OK 本节结束，是不是很乱？

乱很正常。那这一节就让它先乱着，下一节我们将请另一个老朋友来帮忙，把它理清楚。这位老朋友就是——instanceof。

具体内容，请看下节分解。

### 5. 深入理解javascript原型和闭包（5）——instanceof

又介绍一个老朋友——instanceof。

对于值类型，你可以通过typeof判断，string/number/boolean都很清楚，但是typeof在判断到引用类型的时候，返回值只有object/function，你不知道它到底是一个object对象，还是数组，还是new Number等等。

这个时候就需要用到instanceof。例如：

![图12：][12]

上图中，f1这个对象是被Foo创建，但是“f1 instanceof Object”为什么是true呢？

至于为什么过会儿再说，先把instanceof判断的规则告诉大家。根据以上代码看下图：

![图13：][13]

Instanceof运算符的第一个变量是一个对象，暂时称为A；第二个变量一般是一个函数，暂时称为B。

Instanceof的判断队则是：沿着A的__proto__这条线来找，同时沿着B的prototype这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回true。如果找到终点还未重合，则返回false。

按照以上规则，大家看看“ f1 instanceof Object ”这句代码是不是true？ 根据上图很容易就能看出来，就是true。

通过上以规则，你可以解释很多比较怪异的现象，例如：

![图14：][14]

这些看似很混乱的东西，答案却都是true，这是为何？

正好，这里也接上了咱们上一节说的“乱”。

上一节咱们贴了好多的图片，其实那些图片是可以联合成一个整体的，即：

![图15：][15]

看这个图片，千万不要嫌烦，必须一条线一条线挨着分析。如果上一节你看的比较仔细，再结合刚才咱们介绍的instanceof的概念，相信能看懂这个图片的内容。

看看这个图片，你也就知道为何上面三个看似混乱的语句返回的是true了。

问题又出来了。Instanceof这样设计，到底有什么用？到底instanceof想表达什么呢？

重点就这样被这位老朋友给引出来了——继承——原型链。

即，instanceof表示的就是一种继承关系，或者原型链的结构。请看下节分解。

### 6. 深入理解javascript原型和闭包（6）——继承

为何用“继承”为标题，而不用“原型链”？

原型链如果解释清楚了很容易理解，不会与常用的java/C#产生混淆。而“继承”确实常用面向对象语言中最基本的概念，但是java中的继承与javascript中的继承又完全是两回事儿。因此，这里把“继承”着重拿出来，就为了体现这个不同。

javascript中的继承是通过原型链来体现的。先看几句代码

![图16：][16]

以上代码中，f1是Foo函数new出来的对象，f1.a是f1对象的基本属性，f1.b是怎么来的呢？——从Foo.prototype得来，因为f1.__proto__指向的是Foo.prototype

访问一个对象的属性时，先在基本属性中查找，如果没有，再沿着__proto__这条链向上找，这就是原型链。

看图说话：

![图17：][17]

上图中，访问f1.b时，f1的基本属性中没有b，于是沿着__proto__找到了Foo.prototype.b。

那么我们在实际应用中如何区分一个属性到底是基本的还是从原型中找到的呢？大家可能都知道答案了——hasOwnProperty，特别是在for…in…循环中，一定要注意。

![图18：][18]

等等，不对！ f1的这个hasOwnProperty方法是从哪里来的？ f1本身没有，Foo.prototype中也没有，哪儿来的？

好问题。

它是从Object.prototype中来的，请看图：

![图19：][19]

对象的原型链是沿着__proto__这条线走的，因此在查找f1.hasOwnProperty属性时，就会顺着原型链一直查找到Object.prototype。

由于所有的对象的原型链都会找到Object.prototype，因此所有的对象都会有Object.prototype的方法。这就是所谓的“继承”。

当然这只是一个例子，你可以自定义函数和对象来实现自己的继承。

说一个函数的例子吧。

我们都知道每个函数都有call，apply方法，都有length，arguments，caller等属性。为什么每个函数都有？这肯定是“继承”的。函数由Function函数创建，因此继承的Function.prototype中的方法。不信可以请微软的Visual Studio老师给我们验证一下：

![图20：][20]

看到了吧，有call、length等这些属性。

那怎么还有hasOwnProperty呢？——那是Function.prototype继承自Object.prototype的方法。有疑问可以看看上一节将instanceof时候那个大图，看看Function.prototype.__proto__是否指向Object.prototype。

原型、原型链，大家都明白了吗？





```

``` 


  [1]: http://ot0v1xy2c.bkt.clouddn.com/javascript001.png
  [2]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [3]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [4]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [5]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [6]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [7]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [8]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [9]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [10]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [11]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [12]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [13]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [14]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [15]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [16]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [17]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [18]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [19]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
  [20]: https://images0.cnblogs.com/blog/138012/201409/182015334711671.png
