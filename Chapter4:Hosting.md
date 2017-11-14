现在，你应该已经对作用域以及变量如何绑定到各级作用域已经非常清楚了。方法作用域和块作用域都遵循这个规则：任何变量声明都会绑定到所在的作用域中。



但是作用域运作和其中的变量的声明之间有个微妙的细节，我们接下来就来讨论它。



### Chicken Or The Egg?

一般人会误认为JavaScript程序是按从上到下逐行理解的。这基本上是正确的，但是有一种情况并不是这样的。



考虑：

```JavaScript
a = 2;

var a;

console.log( a );
```

你觉得`console.log(..)`会打印出什么呢？

很多开发者会认为是`undefined`，因为`var a`在`a = 2`之后，他们就认为变量被重新定义了，因此被赋以缺省值`undefined`。然而，正确答案是`2`；



考虑另一段代码：

```javascript
console.log( a );

var a = 2;
```

通过之前的代码你知道了有些代码不是完全从上往下运行的，所以你可能会以为这段代码运行的结果也会打印`2`。有些人会觉得，由于`a`在声明前被使用，所以应该会报`ReferenceError`异常。



然而，这两种猜测都不对。结果应该是`undefined`。



那么，到底发生了什么呢？这就会出现先有鸡还是先有蛋的问题。到底哪个先，是声明（蛋）还是赋值（鸡）？



### The Compiler Strikes Again

为了回答这个问题，我们需要回顾一下第一章以及对编译器的讨论。引擎在解释JavaScript代码前会编译它。编译的部分工作就是给所用声明寻找并分配到合适的作用域。第二章告诉我们，这就是词法作用域的核心部分。



因此，所有的声明，包括变量和方法，在代码运行前就处理好了。



当你看到`var a =2;`时，你可能会把它看做一个表达式。但是JavaScript把它看做两个表达式：`var a;`和`a = 2`；第一个表达式，也就是声明，在编译阶段就被处理了。第二个表达式，也就是赋值，会被留在原地等待执行。



上述的第一个代码片段实际上应该是这样子被处理的：

```JavaScript
var a;
```

```JavaScript
a = 2;
console.log( a );
```

第一部分是编译，第二部分是运行。



第二个代码片段也是类似的：

```JavaScript
var a;
```

```JavaScript
console.log( a );
a = 2;
```

因此，打个比方，变量和方法的声明从它们出现的地方被挪到了代码的最顶部。这就叫做“提升”。



那么，也就是说蛋（声明）在鸡（赋值）的前面。



**注意：**只有声明被提升了，其他的赋值等其他运行逻辑会留在原地。如果提升可以改变代码的执行顺序，那将会是一场灾难。

```javascript
foo();

function foo() {
	console.log( a ); // undefined

	var a = 2;
}
```

`foo`方法的声明被提升了，因此第一行代码中的方法调用能够被运行。



需要注意的是，提升在每个作用域都在发生。尽管前面大部分的代码片段已经经过简化了，直接被包含在全局作用域里了，但是在`foo(..)`里的`var a`是被提升到`foo(..)`的顶部的（明显不是被提到了整个程序的顶部）。因此，这段程序可以更加精确地解析为：

```JavaScript
function foo() {
	var a;

	console.log( a ); // undefined

	a = 2;
}

foo();
```

就像我们已经说过的，方法的声明也被提升了。但是，方法表达式并没有。

```javascript
foo(); // not ReferenceError, but TypeError!

var foo = function bar() {
	// ...
};
```

变量标识符`foo`被提升到了所在的作用域的顶部，因此`foo()`没有报`ReferenceError`的异常。但是`foo`此时还没有值（如果它是个方法声明而不是表达式，那才会有值）。因此，`foo()`就是对`undefined`进行函数调用，就会引起`TypeError`异常。



另外，如果这是个具名方法表达式，在作用域中也是无法获取到这个名字的标识符的：

```JavaScript
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
	// ...
};
```

这段代码可以解析为：

```JavaScript
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
	var bar = ...self...
	// ...
}
```

#### 方法优先

方法声明和变量声明都会被提升。不过，方法声明的提升优先级高于变量。考虑：

```JavaScript
foo(); // 1

var foo;

function foo() {
	console.log( 1 );
}

foo = function() {
	console.log( 2 );
};
```

引擎是这样解读代码的：

```JavaScript
function foo() {
	console.log( 1 );
}

foo(); // 1

foo = function() {
	console.log( 2 );
};
```

注意，`var foo`是重复的声明，因此被忽略了，尽管它在`function foo()…`的前面。因为方法声明的提示相比一般变量优先级要高。



尽管`var`声明被忽略了，但是后来的方法声明依然可以覆盖前面的方法声明。

```JavaScript
foo(); // 3

function foo() {
	console.log( 1 );
}

var foo = function() {
	console.log( 2 );
};

function foo() {
	console.log( 3 );
}
```

虽然这些学术理论听起来挺无聊的，但是它表明一个问题：在同一个作用域中的重复的定义是非常糟糕的，会导致一些令人困惑的结果。



出现在常规的块中的方法声明会被提升到所在作用域，而不是像下面这样被控制提升。

```JavaScript
foo(); // "b"

var a = true;
if (a) {
   function foo() { console.log( "a" ); }
}
else {
   function foo() { console.log( "b" ); }
}
```

然而，需要重点注意的是，这个行为是不可靠的，有可能在未来的JavaScript版本汇总被改变，所以，最好避免在块中声明方法。



### Review (TL;DR)

我们可能会以为`var a = 2;`是一个表达式，但是JavaScript引擎不这么看。它将其看作两个表达式：`var a`和`a = 2`，第一个表达式是编译阶段的任务，第二个是在运行阶段的任务。



这就会导致一个作用域中所有声明，不管它们在哪里出现，它们都会在代码运行前被处理。这就好像声明（变量和方法）被移到了它们所在的作用域的顶部，这就是”提升“。



声明本身被提升了，但是赋值，包括方法表达式的赋值并没有被提升。



小心重复声明，尤其是常规的var声明和方法声明混杂在一起的。如果你要这么写代码，危险就在不远处等着你。

