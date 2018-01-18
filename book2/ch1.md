JavaScript中最令人困惑的机制之一就是`this`关键词。它是一个特殊的关键词，它会在每个函数作用域中自动地被定义。但是即使是老练的JavaScript开发者也不清楚它指向的是什么。



> 任何足够先进的技术都和魔法一样。——Arthur C. Clarke



JavaScript的`this`机制没有那么先进，但是依然有很多程序员觉得它很复杂很难以理解。同样的，如果你对`this`也没有清晰的理解的话，你也会觉得它很魔幻。



**注意：** “this”是沟通过程中极其常见的一个代词。所以，在交流过程中很难区“this”是代词还是关键字。清晰起见，我总会一直使用`this` 表示关键字，使用“this”或者 this 来表示代词。（中文不必区分，不用理这段）。



## Why `this`?

如果`this`机制甚至是对熟练的JavaScript程序员都这么难懂，那它还有用么？是更有价值还是更麻烦的机制？在我们讲怎么用之前，先来了解下为什么要用它。



现在阐述下`this`的用处：

```JavaScript
function identify() {
	return this.name.toUpperCase();
}

function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER			
```

如果看不懂这段代码，请不要担心！你很快就会明白的。把心中的疑问放到一边，我们先来理解为什么这么做。



这段代码让`identify()`和`speak()`函数能直接在不同的上下文对象（`me`和`you`）中使用，而不需要为每个对象生成独立的版本。



除了依赖`this`,你也可以显示地把上下文对象传入`identify()`和`speak()`函数中。

```JavaScript
function identify(context) {
	return context.name.toUpperCase();
}

function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```

`this`机制提供了更优雅的隐式传递对象的方式，让API更简洁，让重用更简单。



随着你使用的模式越来越复杂，显示地传递上下文对象让代码变得越来越杂乱，而`this`就不会。当我们研究对象和原型时，函数能自动地引用合适的上下文对象是多么地有用。



## Confusions

我们马上就会要就`this`到底是怎么工作的，但是首先我们必须消除对它的一些误解。



当程序员试图从字面意思理解它的时候，"this"这个单词本身就会天然地形成误解。它经常被误解为以下两种意思。



### Itself

第一个最常见的误解就是，把`this`当做函数本身的引用。不过，至少从语法上这么推断是没错的。



但是你为什么会想在函数内部引用它自己呢？最可能的原因应该就是递归（在函数内部调用它自己）或可以在第一次调用时就解绑自己的事件处理器。



不熟悉JS机制的程序员可能会认把函数作为对象（JavaScript中的所有函数都是对象！）来引用可以在两个函数的调用之间存储状态（属性的值）。当然这是可行有用的，不过本书接下来会介绍比函数对象更好的存放状态的模式。



不过，我们先来看下为什么`this`不是让函数获取自身的引用的。



考虑以下代码，我们来跟踪下函数`foo`被调用了多少次：

```JavaScript
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```

`foo.count`依然是`0`，即使`console.log`表达式清楚地表明`foo(..)`被调用了4次。所以从字面量来理解`this`是不对的（`this.count++`）。



当代码运行到`foo.count =0 `时，事实上是给函数对象`foo` 添加了一个`count`属性。但是对函数内部引用的`this.count`来说，`this`完全不是指向函数对象的，所以即使属性名是相同的，但根对象是不同的，因此产生了困惑。



**注意： **负责的程序员可能会问，“这个不断增长的`count`既然不是我想要的那个，那到底是哪个`count`在增长呢？”事实上，当你深挖下去的时候，你会意外地发现，你创建了一个全局变量`count`（具体发生了什么请看第2章），并且它的值是`NaN`。当然，当你知道了这个神奇的结果，你又会问：“为什么它是全局的，为什么它最后的值是`NaN`而不是合适的数值？”（见第2章）。



很多程序员可能不愿意继续深挖为什么`this`引用表现的和预期的不一样，不愿意尝试解决这个棘手但重要的问题，而是立刻想用其他方法直接解决这个问题，比如，创建一个对象来持有`count`属性：

```JavaScript
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	data.count++;
}

var data = {
	count: 0
};

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( data.count ); // 4
```

当然这个方法确实“解决”了问题，但是它却回避了真正的问题——没有理解`this`的意义和它是怎么工作的——取而代之的是，回到舒适地带：词法作用域。



**注意： **词法作用域是一个非常完美又实用的机制。我不是贬低使用它（请见本系列的《作用域与闭包》）。但是总是猜测怎么使用`this`，并总是猜错，不是你逃避了解`this`，用词法作用域来解决问题的理由。



为了从函数对象内部引用它自身，只使用`this`是不够的。一般来说，你应该通过一个指向函数对象的词法标识符（变量）来引用。



考虑以下两个函数：

```JavaScript
function foo() {
	foo.count = 4; // `foo` refers to itself
}

setTimeout( function(){
	// anonymous function (no name), cannot
	// refer to itself
}, 10 );
```

第一个函数是具名函数，`foo`就是可以在函数内部指向函数本身的引用。



在第二个函数中，传入`setTimeout(..)`函数是匿名函数，所以没有比较好的方法来引用函数对象本身。



**注意： **老派又过时又难懂的`arguments.callee`引用也能在函数对象内部指向正在运行的函数。这个引用是唯一一个能在匿名函数对象内部引用自身的方法。然而，最好的方法是，任何情况下都避免使用匿名函数，至少那些需要引用自身的函数都要使用具名函数。`arguments.callee`已经被舍弃了，最好不要使用了。（*daisy注：*ES5的标准模式已经禁用了，原因是会影响现代浏览器的性能，arguments不能被尾递归和内联优化，影响闭包）



因此，我们这个例子的另一个解决办法就是在每个地方都使用`foo`标识符作为函数对象的引用，完全不要使用`this`，就像这样：

```JavaScript
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	foo.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

然而，这种办法同样又绕开了对`this`的理解，完全依赖于变量`foo`的词法作用域。



这个问题还有一种解决方法，就是强制`this`指向`foo`函数对象：

```javascript
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	// Note: `this` IS actually `foo` now, based on
	// how `foo` is called (see below)
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		// using `call(..)`, we ensure the `this`
		// points at the function object (`foo`) itself
		foo.call( foo, i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

**我们不再回避`this`，而是直面它。 ** 如果你现在对这个技术有点迷糊，请不要担心，我待会儿会完整地解释它。



### Its Scope

对`this`的另一个常见的误解就是认为，它是指向函数作用域的。这是个棘手的问题，因为从某种层面上来说，这是对的，但是从另个方面来说，又是错的。



明确地说，`this`在任何情况下都不指向函数的**词法作用域**。的确，在函数内部，作用域有点像对象，可见的标识符就像它的属性。但是作用域这个“对象”无法通过JavaScript代码获取到，它是引擎内部的一部分。



考虑以下代码，它试图（不过失败了）跨过边界，并使用`this`隐式地指向函数的词法作用域：

```JavaScript
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); //undefined
```

这段代码里的错误不止一个。虽然这段代码看起来好像是我们故意写出来的例子，但是实际上它是我们精简了一段来自公共社区中的互助论坛的代码。这段代码完美地展示了`this`是多么的误导人。



首先，`this.bar()`试图引用`bar()`函数。这确实是会起效，我们稍后来解释。调用`bar()`最自然的方式就是去掉前面的`this.`，直接使用标识符的词法引用就好了。



然而，这么写代码的程序员其实是想要通过`this`在`foo()`和`bar()`的词法作用域之前搭建一座桥梁，这样的话，`bar()`就可以访问`foo`内部作用域的变量`a`了。**这样的桥梁是不存在的。 **你不可能使用`this`	在一个词法作用域中找到任何东西的。



每次你感觉你要把词法作用域的查找和`this`混淆时，记得提醒自己：他们之间没关系。



## What's `this`?	

把各种误解放到一边，现在让我们把注意力放到`this`机制到底是怎样的。



我们之前说过，`this`不是在书写阶段绑定的，而是在运行时绑定的。它的上下文取决于函数调用的环境。`this`和函数在哪里声明的无关，和函数被怎么调用的有关。



当一个函数被调用时，一个活动记录，或者说一个执行上下文，被创建了。这个记录记载了函数被调用的地方（调用栈）、函数是怎么被调用的以及哪些参数传递了过去等等信息。这个记录的其中一个属性就是`this`。在函数运行的整个期间，`this`都会被使用。



在下一章，我们将会学习如何找到函数调用的位置，从而知道函数执行时是怎么绑定`this`的。



## Review (TL;DR)

`this`绑定对那些不花时间了解这个机制的工作原理的程序员来说，永远都是个困惑。猜测、试错和盲目地从 Stack Overflow复制粘贴，并不能让你真正地理解`this`机制。



想知道`this`是什么，首先，你得知道`this`不是什么，尽管你可能误解过。`this`既不是函数自身的一个引用，也不是函数作用域的一个引用。



`this`是函数被调用时的一个绑定，它指向什么完全取决于函数是在哪里被调用的。



