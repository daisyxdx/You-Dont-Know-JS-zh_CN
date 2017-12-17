正如我们在第二章中所探讨的那样，作用域包含了一系列的“泡泡”。它们就像容器一样盛放着标识符（变量、方法）的声明。这些泡泡整齐地一个一个嵌套起来，并且这些嵌套关系在代码撰写时就被确定好了。



但是到底是什么东西制造了一个新的泡泡呢？仅仅是方法么？JavaScript中还有其他结构能创造这样的作用域泡泡么？



#### Scope From Functions

这些问题的的最常见的答案就是：JavaScript的作用域是基于方法的。也就是说，你所声明的每个方法都为它自己创建了一个作用域泡泡，但是其他别的结构都不能创建他们自己的作用域泡泡。但是，只要我们再深入一些了解就知道，这种说法是不正确的。



但是首先，我们还是先来探讨下方法的作用域和它的含义。



考虑：

```javascript
function foo(a) {
	var b = 2;

	// some code

	function bar() {
		// ...
	}

	// more code

	var c = 3;
}
```

在这个代码片段中`foo(..)`的作用域泡泡包含了标识符`a`，`b`，`c`和`bar`。不管标识符是在哪里声明的，这个标识符所代表的变量或方法都是属于这个作用域泡泡的。我们将会在下一章来具体探讨这种机制。



`bar(..)`拥有它自己的作用域泡泡。就像全局作用域也拥有自己的作用域泡泡，不过它只有一个标识符：`foo`。



因为`a`,`b`,`c`和`bar`都是属于`foo(..)`下的作用域泡泡的，在`foo(..)`的外部是获取不到它们的。也就是说，下面这些代码都会产生`ReferenceError`类型的异常，因为在全局作用域中这些标识符是获取不到的。

```javascript
bar(); // fails

console.log( a, b, c ); // all 3 fail
```

在`foo（..)`里这些标识符都是可获取的，在`bar(..)`里也是可获取的（假定`bar(..)`里面没有遮蔽这些标识符的声明）。



在方法作用域中，所有的变量都是属于方法的，它们在整个方法中都是可以被使用和重用的（嵌套在方法中的作用域也可以使用）。这种设计是非常有用的，能充分利用JavaScript的变量根据需要改变值额类型的这种“动态”特性。



另一方面，如果你在使用存在于整个作用域的变量时不加注意的话，可能会引起一些意想不到的问题。



### Hiding In Plain Scope

常规的使用方法的做法就是：先声明一个方法，然后在它里面加代码。但是反过来做同样也是很有用的：把任意的一段已经写好的代码用方法声明包起来，就好像把代码藏起来了一样。



这种做法的结果就是：给这段代码添加了一个作用域泡泡。那也就是意味着这段代码里面的声明（变量和方法）是和这个新的方法联系在一起的，而不是之前所在的那个作用域了。换句话说，你可以把变量和方法藏在这个方法作用域里面了。



藏变量和方法的技巧有啥用呢？



有很多原因促成了基于作用域的隐藏做法。它们主要源自于软件设计中的原则：“最小特权原则”，也可以叫做“最小授权原则”或“最少暴露原则”。这个原则是指：在软件设计的过程中，对模块和对象的API来说，应该只暴露出一些必须要暴露的内容，并将其他一切都藏起来。



这个原则在这里就具体为：如何选       择作用域来包含变量和方法。如果所有的变量和方法都存在于全局作用域中，那他们任何的作用域中都可以获取到。但是这样做是违背“最少暴露原则”的。因为你把很多本应该是私有的变量和方法暴露了出来。



例：

```JavaScript
function doSomething(a) {
	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

function doSomethingElse(a) {
	return a - 1;
}

var b;

doSomething( 2 ); // 15
```

在上述代码片段中，变量`b`和方法`doSomethingElse(..)`应该是`doSomething(..)`私有的。给外部作用能够获取到`b`和`doSomethingElse(..)`的权限不仅是没必要的，同时也是危险的。因为，它们有可能会被故意或无意地错误地使用，从而在`doSomething(..)`中变得不适用了。



更好地代码设计：将`doSomething(..)`的私有变量放到它自己的作用域中，如下：

```javascript
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

doSomething( 2 ); // 15
```

现在，`b`和`doSomethingElse(..)`就不再会受到`doSomething(..)`外部的作用域的影响了。功能和最终结果没有收到影响，但是具体内容私有化了，好的程序设计都会考虑这么做。



#### Collision Avoidance（避免碰撞）

隐藏变量和方法的另一个好处是：避免两个不同拥有相同名字不同用处的标识符相撞。相撞经常会导致值被意外重写。



例：

```JavaScript
function foo() {
	function bar(a) {
		i = 3; // changing the `i` in the enclosing scope's for-loop
		console.log( a + i );
	}

	for (var i=0; i<10; i++) {
		bar( i * 2 ); // oops, infinite loop ahead!
	}
}

foo();
```

`bar(..)`中的`i=3`表达式在`foo(..)`中的for循环中被意外地重写了。在这种情况下，`i`会一直被设置成`3`，也就是说`i`会一直`<10`，就会导致无限循环的发生。



`bar(..)`中的表达式需要一个本地变量才能较好的使用，这个变量的名字是啥都行。`var i = 3;`这样写就可以解决这个问题了（这样写创建的`i`就是使用之前提到“变量遮蔽”的技巧）。你也可以用别的名字来命名这个标识符，比如`var j = 3;`这样。但是有时如果软件设计可能会比较自然地使用相同的标识符名，那么利用作用域来隐藏内部声明会是唯一最佳实践。



#### Global "Namespaces"

一个特别典型的变量冲撞的例子发生在全局作用域中。多个库导入你的项目中，但是它们却没有很好的隐藏自己的内部变量和方法的话，它们彼此之间就很可能会产生冲撞。



这些库通常都会在全局作用域中创建一个名字足够独特的变量，通常是个对象。这个对象就被用来当做这个库的“命名空间”。所有的具体暴露的功能都作为它的属性了，而不是在顶级的词法作用域中自己来声明了。



例：

```javascript
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function() {
		// ...
	},
	doAnotherThing: function() {
		// ...
	}
};
```

#### Module Management

另一个避免冲撞的的方法是模块管理。使用这些管理工具，任何库都不能再全局作用域中添加标识符，但是它们会通过依赖管理机制将自己本身被导入到另一个特定的作用域中。



显而易见，这些工具并没有能够违反词法作用域规则的神奇魔法。它们只是单纯地使用了作用域的规则，来做到没有标识符注入到共享的作用域中——标识符只能存在于私有的不会发生冲撞的作用域中，这样就能保证不会有意外地冲撞发生。



### Functions As Scopes

我们已经知道了，可以用方法将任意的代码片段包裹起来，这可以有效地将变量和方法声明藏起来。



例：

```JavaScript
var a = 2;

function foo() { // <-- insert this

	var a = 3;
	console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```

虽然这种技术可以起到一定作用，但是这不是非常好的办法。这样做会存在一些问题。第一，我们要为此声明一个具名函数`foo()`，那么这个名称`foo`本身就污染了外部作用域（在这里就是全局作用域）。第二，我们如果要执行它，就必须显式地通过方法名`foo`来调用它。



如果方法不需要具名（或者方法名不会污染外部的作用域也行）且能立即执行就更好了。



幸运的是，JavaScript提供了一个能解决上述问题的方法。

```JavaScript
var a = 2;

(function foo(){ // <-- insert this

	var a = 3;
	console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

让我们来分析下这段代码发生了什么。



首先，被方法包裹的那段代码是以`(function...`开始的，而不是`function...`。这看似极小的细节，但确是非常重要的。这使得这段不再是一个标准的方法声明，而是一个方法表达式。



**注意：**区分声明和表达式最简单的办法就是看“function”这个关键字出现的位置。如果“function”在最前面，那么就是方法声明；否则就是方法表达式。



方法声明和方法表达式之间的关键区别就是：标识符名字是在什么时候被绑定的。



比较值钱的两段代码。第一段代码中，`foo`这个名字实在外部作用域中被绑定的，我们可以直接用`foo()`来调用它。第二段代码，`foo`不是在外部的作用域中绑定的，而是在它自己的方法中绑定的。



换句话说，`(function foo(){ .. })`作为一个表达式意味着，`foo`只能在`..`所代表的地方所找到，而不是外部的作用域中。把`foo`这个名字隐藏在它自己所代表的作用域中就不会污染外部的作用域了。



### Anonymous vs. Named

你可能对方法表达式作为传参很熟悉了，例：

```javascript
setTimeout( function(){
	console.log("I waited 1 second!");
}, 1000 );
```

这个是匿名方法表达式，因为`function()...`是没有名字的。方法表达式可以是匿名的；但是JS语法规定方法声明不能没有名字。

匿名方法表达式书写起来很简单快捷，因此，很多库和工具鼓励大家这么写。然而，这样做存在一些需要考虑的缺点：

1.匿名方法在栈跟踪中没有有效的名字，这会使得调试变得困难。

2.没有方法名的话，当方法需要引用自身时，比如递归时，就只能调用过时的`arguments.callee`。另一个需要引用自身的例子是：事件监听器在事件触发后解绑自身。

3.匿名方法省略了，能够使代码更加地具有可读性和易于理解 方法名。一个描述性的方法名能使代码不言自明。



**行内方法表达式**更加强大和有用——不管是匿名还是具名，在这个问题上都是一样的。给方法表达式提供一个名字可以有效地解决上述问题，并且不会产生别的问题。总是给你的方法表达式一个名字是最佳实践：

```javascript
setTimeout( function timeoutHandler(){ // <-- Look, I have a name!
	console.log( "I waited 1 second!" );
}, 1000 );
```



#### Invoking Function Expressions Immediately

```JavaScript
var a = 2;

(function foo(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

将方法表达式放在了一对圆括号里面，在它的后面再添加一对圆括号，我们就可以直接运行这段代码了。形如：`(function foo(){ .. })()`。最外面的那对括号，使得方法称为了表达式最后那对括号，使方法立刻执行。



这种形式非常常见，多年前委员会给了它一个术语：`IIFE`, **I**mmediately **I**nvoked **F**unction **E**xpression的缩写。



当然IIFE是不需要方法名的，IIFE最常见的形式是使用匿名方法表达式。虽然具名的IIFE不常见，但是相比匿名的IIFE，它依然有之前讲的这些优点，因此，具名IIFE依然是一个不错的实践。

```JavaScript
var a = 2;

(function IIFE(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

相比传统的IIFE，有些人有些人喜欢这么用：`(function(){ .. }())`。仔细看两种不同形式的区别。第一种，表达式是包裹在括号里的，调用方法的那对括号是在外面的。第二种，调用方法的括号在第一个括号的里面。



这两种形式在功能上是完全一样的。`只是写法上不同而已`，选择哪个随便你。



IIFE还有一种变化形式，这种形式非常常见，就是在IIFE中传入参数。它是利用了IIFE的本质——方法的调用。

例：

```JavaScript
var a = 2;

(function IIFE( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

上述IIFE中，传入了参数`window`，但是给这个参数明明为`global`。这样，我们及可以清楚的区分`global`引用和非`global`引用。当然，你也可以传入任何你想传入的参数，你也可以将参数命名为任何你觉得合适的名字。



IIFE的另一个用处是：解决`undefined`标识符的默认值可能被不正确地覆写所引起的异常。通过将参数命名为`undefined`，但是给参数传入任何值，我们就可以保证`undefined`标识符的值确实是undefined。

```JavaScript
undefined = true; // setting a land-mine for other code! avoid!

(function IIFE( undefined ){

	var a;
	if (a === undefined) {
		console.log( "Undefined is safe here!" );
	}

})();
```

IIFE的另一个种变化形式：将两者的顺序反过来，要运行的方法放在后面，要传入的参数放在前面。这种模式在UMD项目（Universal Module Definition）中被大量使用。有些人觉得这样写更利于理解，尽管这样写有些啰嗦。

```JavaScript
var a = 2;

(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```

表达式`def`在第二部分被定义，传入的参数（这个参数也叫`def`）在第一部分被定义。最后，参数`def`被调用，并将`widow`作为参数传入，并将其命名为`global`。



### Blocks As Scopes

尽管方法作用域是最常见的作用域单元，并且被广泛应用再在JS的程序设计上，依然存在着其他的作用域单元，有时它们能更好更清晰呈现代码。



很多语言都支持块作用域，因此其他语言的开发者会对这种思维模式比较熟悉，主要从事JavaScript开发的人可能会这个概念有些陌生。



但是即使你从未写过块作用域，你应该对以下这种形式非常熟悉了：

```JavaScript
for (var i=0; i<10; i++) {
	console.log( i );
}
```

直接在for循环的开头定义`i`，通常是因为只是现在这个for循环里面使用`i`，并且忽略了这个变量实际上是在外部作用域中的。



这就是块作用域的所有内容：变量在哪里使用，就尽量在其附近声明它。例：

```JavaScript
var foo = true;

if (foo) {
	var bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}
```

`bar`变量只在if里面使用，因此我们只需要将其声明在if块内部就可以了。然而，当我们使用`var`声明变量时，不管在哪里声明，它都是全局变量。这段代码其实是伪装成块作用域，只是为了读者能够清楚地了解快作用域。



块作用域是“最少暴露原则”一种实践：将代码的信息隐藏在块里面。

再次考虑for循环：

```JavaScript
for (var i=0; i<10; i++) {
	console.log( i );
}
```

为什么要把只在for循环中用到的`i`变量污染到整个作用域中呢？



更重要的是，开发者需要自己去检查变量在别的地方是不是被意外地使用了，比如在错误的地方使用一个未知的变量就会引起异常。块作用域就会让`i`只在for循环内使用，如果在其他地方使用就会引起异常。这使得变量不会被混乱地重用，也不会让变量变得很难维护。



但是很遗憾，JavaScript没有使用块作用域的机制。



除非你更深入地研究。



##### `with`

我们已经在第二章中了解了`with`了。尽管难理解它，但是它确实是块作用域的一个例子。对象确实只能在由`with`产生的作用域中存在。



##### `try/catch`

很少有人知道，JavaScript的ES3规范中规定，`try/catch`的`catch`分支中声明的变量只在`catch`块中有效。（埖埖大雾：只有err符合，其他在catch里面创建的变量并不是块作用域变量）

例：

```JavaScript
try {
	undefined(); // illegal operation to force an exception!
}
catch (err) {
	console.log( err ); // works!
}

console.log( err ); // ReferenceError: `err` not found
```

正如你所见，`err`只存在于`catch`分句中，当你在别的地方印用它的时候就会报错。



**注意：**尽管这个行为已经被标准化了，并且被大部分的标准 JavaScript 环境(除了老版本的 IE 浏览器)所支持，但是当同一个作用域中的两个或多个 catch 分句用同样的标识符名称声明错误变量时，很多静态检查工具还是会发出警告。实际上这并不是重复定义，因为所有变量都被安全地限制在块作用域内部，但是静态检查工具还是会很烦人地发出警告。



为了避免不必要的警告，一些开发者会将其命名为`err1`、`err2`等。有些开发者会直接关闭静态检查器对重复变量名的检查。



`catch`的快作用域看起来似乎没啥用，但是看附录B就会发现一些很有用的信息。



##### `let`

到目前为止，我们看到JavaScript在块作用域的使用上只有一些奇怪的小众的机制。如果只能使用这些机制的话，JavaScript中的块作用域对开发者来说就没有太大的用处。



幸运地是，ES6改观了这一问题，引入了一个全新的关键词`let`，和`var`一样，`let`也是用来声明变量的。



关键词`let`把变量声明和作用域绑定在一起（通常是`{..}`内部）。

```JavaScript
var foo = true;

if (foo) {
	let bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}

console.log( bar ); // ReferenceError
```

用`let`将变量隐式地绑定在一个已存在的作用域中。如果你在书写代码时并没有留意各个作用域包含了哪些变量，并且随意地挪动它们，那么就会造成混乱。



为块作用域显式地创建块可以部分解决这个问题，使变量的附属关系变得更加清晰。通常来讲，显式的代码优于隐式和不清晰的代码。显示的块非常好写，并且和其他语言的块作用域的工作原理一致。

```JavaScript
var foo = true;

if (foo) {
	{ // <-- explicit block
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```

我们可以随意地为`let`创建一个块，只需要一对括号就够了。在上述代码中，在if声明内部显示地创建了一个块。如果代码要重构的话，只要把括号内的代码一起移动就好了。



**注意：**另一种显示块作用域的写法，请看附录B。



在第4章，我们将会讨论“提升”，提升的声明将会在它所在的整个作用域中存在。



然而，`let`的声明并不会提升到整个作用域。声明的代码在运行之前，是不会存在的。

```JavaScript
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

##### Garbage Collection

块作用域的闭包性质还有利于垃圾回收。闭包机制将会在第5章着重讲解，我们在这里只简短地说明。



考虑：

```JavaScript
function process(data) {
	// do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

处理点击回调事件的`click`方法根本不需要`someReallyBigData`变量。所以，从理论上来讲，`process(..)`运行之后，占很多内存的数据结构就可以被当做垃圾回收了。然而，因为`click`方法的闭包囊括了整个作用域，因此JS引擎仍然要保持着这个结构。



块作用域能解决这个问题，让引擎清楚地知道已经不需要保持`someReallyBigData`了：

```JavaScript
function process(data) {
	// do something interesting
}

// 这个块下面的代码，在运行过后都可以被回收！
{
	let someReallyBigData = { .. };

	process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );			
```

显示地声明块，并将变量绑定于此是非常有用的。

##### `let` Loops

`let`在for循环里表现非常好。

```JavaScript
for (let i=0; i<10; i++) {
	console.log( i );
}

console.log( i ); // ReferenceError	
```

`let`不仅把`i`从for循环的头部绑定到了for循环的内部，而且还使得`i`在每次迭代时都被重新绑定了，因此在每次循环迭代之后都能对`i`重新赋值。



下面是另一种解释每次迭代都会重新绑定的方式：

```JavaScript
{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // re-bound for each iteration!
		console.log( i );
	}
}		
```

每次迭代都会重新绑定的原因是很有意思的，我们将会在第5章讨论闭包时搞清楚这个原因。



由于`let`声明是将变量绑定在任意的块中，而非整个方法作用域中，因此，当代码中存在对`var`的隐式依赖时，就会有很多陷阱。如果将`var`替换为`let`，那以后要重构的话就要小心一点了。

考虑：

```JavaScript
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	if (baz > bar) {
		console.log( baz );
	}

	// ...
}	
```

这段代码很容易这样重构：

```JavaScript
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	// ...
}

if (baz > bar) {
	console.log( baz );
}
```

但是，如果使用了块作用域变量的话，就得小心了：

```JavaScript
var foo = true, baz = 10;

if (foo) {
	let bar = 3;

	if (baz > bar) { // <--移动的时候，不要忘了`bar`
		console.log( baz );
	}
}
```

附录B中给出了一种稳健的方便重构的块作用域形式。



##### `const`

除了`let`，ES6还加入了`const`。`const`也能创建块作用域变量，但是它的值是常量。在声明后改变它的值，会引起异常。

```JavaScript
var foo = true;

if (foo) {
	var a = 2;
	const b = 3; // block-scoped to the containing `if`

	a = 3; // just fine!
	b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```



### Review(TL;DR)

在JavaScript中，方法是最常见的作用域。声明在在某方法中的变量和方法是隐藏在这个方法作用域中的，这是为了软件的优良性而设计的原则。



但是方法不是唯一的作用域单元。块作用域是指，变量和方法可以属于任意的块（一般来说，就是任意的大括号对`{..}`）。



从ES3开始，`try/catch`结构的`catch`分支拥有块作用域。



在ES6，`let`关键词被引用，用来在任意的代码块中声明变量。`if (..) { let a = 2; }` 将会声明一个绑定在`if`的 `{ .. }`块中变量。



很多人认为，块作用域不应该完全地替代方法作用域，两种功能应该共存，开发者应根据实际情况挑选使用这两者以使程序具有良好的可读性和可维护性。































































