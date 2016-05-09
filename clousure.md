# 闭包(Closure)

## 概念

在编程语言中，闭包（英语：Closure），又称词法闭包（Lexical Closure）或函数闭包（function closures），是实现`词法作用域变量名绑定`（lexically scoped name binding）的一种技术。是`函数体`和`引用环境`的组合成的实体。

`函数体`：执行语句集合。

`自由变量`：在函数体外定义，函数体内使用的变量，即外部定义的变量。

`引用环境`：保存了 函数体里所有`自由变量`（free variable） 与 它们的值或存储位置 的映射。

一个闭包，允许`函数体`通过`引用环境`以`自由变量`的形式去访问*捕获的变量*（在词法上不属于该函数作用域内的外部变量）。

## 表现形式

来看看例子

	function somescope(){
		var foo,bar;
		// ...
		
		function fun(){
			console.log(foo+bar);
			// ...
		}
		// ...
		return fun;
	}
	
	var closure1 = somescope();
	var closure2 = somescope();

在执行了`somescope`之后，`closure1`和`closure2`就是JavaScript实现的`闭包`了。`return`将`闭包`传送出了`词法作用域`，使得它的生存期并没有终止于`词法作用域`。那么这个时候`closure1`和`closure2`包含了哪些东西呢？简单来说有fun的函数体，还有就是`somescope`里的变量`foo`和`bar`当时的值。`foo``bar`的值就代表了`closure1``closure2`的`引用环境`。

> *作为闭包实现的函数能作为函数返回值返回，意味着被返回的函数需要用到的外部定义的变量的值或引用位置都会被像快照一样保存下来。*

闭包并不就是函数，JavaScript函数是闭包的一种实现。闭包从词义上来讲，也就是闭合打包了一部分需要用到外部变量的值或引用位置。

## 实现机制

### 作用域链

JavaScript的函数在执行时会创建一个`执行上下文`，这个`执行上下文`有一个重要对象，该对象保存了该函数的所有局部变量（local variable），就叫`变量对象`，所有局部变量都是这个变量对象的属性。既然包含了所有的局部变量，那么这个`变量对象`就可以看作是该函数的`局部作用域`，函数体内出现的所有的局部变量名都可以通过这个局部作用域查找到相应的局部变量值。

执行上下文除了保存当前函数的`变量对象`，还引用了外层函数的`变量对象`，这些`变量对象`通过执行上下文传来成了`作用域链`，JavaScript解释引擎在查找变量名对应的变量值的时候，就是通过这个作用域链一层一层往上查，一直到全局作用域，如果到链的顶端都查找不到，则会抛出变量未定义错误（Undefined Error）。

`词法作用域变量名绑定`需要变量的作用域覆盖所有的内嵌函数，以及所有的递归内嵌。显然JavaScript作用域链机制是符合`词法作用域变量名绑定`的。

上面提到的`变量对象`作为`局部作用域`构成的`作用域链`其实就是闭包概念里的`引用环境`。

## 闭包与函数

在我认为，闭包就像是一个可以配置的函数，原先函数的所有`参数`都必须定义在`行参列表`里，但是闭包还可以将某些`参数`放入`引用环境`中。这能够引发出一个更有意思的编程模式。我们可以利用`引用环境`来存储函数计算的中间结果以传递给下层嵌套的函数。

参考一些代码：

	function something(){ console.log('do something!') }
	function anotherthing(){ console.log('do anotherthing!') }
	function Person(doWork){
		this.doWork = doWork; 
	}
	Person.prototype.Work(){
		this.doWork();
	}
	
	var p1 = new Person(something);
	var p2 = new Person(anotherthing);

利用闭包特性的改写：

	function something(){ console.log('do something!') }
	function anotherthing(){ console.log('do anotherthing!') }
	function CreatePerson(doWork){
		function Person(){
			this.doWork = doWork; 
		}
		return Person;
	}
	var Person1 = CreatePerson(something);
	var Person2 = CreatePerson(anotherthing);
	
	var p1 = new Person1();
	var p2 = new Person2(); 

试想一下如果需要用到Person1和Person2的地方有非常多的情形下，要少写多少次`something`和`anotherthing`。

`闭包`更像一个`函数模板`的`实例`。`闭包`这一概念的提出，将代码与数据之间的差异进一步的模糊化。在编程语言里，函数以闭包的形式出现，使得函数和对象基本没有差别(函数成为一等公民：first class)，闭包保留了函数原先的处理数据的功能，只不过在闭包里面，处理数据的代码变成了`函数体`这种内部数据，并且`函数体`还可以使用`引用环境`这种动态存储区。

有了`闭包`，巧妙地利用`函数体`和`引用环境`，可能会带来许多不一样的编程技巧。最直观的一个特性就是我们又多了一种传递变化参数的方式。

## 参考资料

+ [wiki][wiki]

[wiki]: https://en.wikipedia.org/wiki/Closure_%28computer_programming%29 "wiki closure"