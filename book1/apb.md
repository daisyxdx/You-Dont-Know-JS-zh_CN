在第3章，我们研究了块作用域。我们了解到至少从ES3发布以后，JavaScript中的`with`和`catch`分句就是块作用域的小例子。



自从ES6引入`let`了，才赋予我们的代码完全的、不受约束的使用块作用域的能力。块作用域在功能上和代码风格上都有令人激动的特性。



但是我们怎么在ES6之前的环境使用块作用域呢？



考虑：

```JavaScript
{
	let a = 2;
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

这段代码在ES6中可以正常工作。但是我们怎么ES6之前的环境中做到呢？`catch`就可以。

```JavaScript
try{throw 2}catch(a){
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

咦~这样写又丑又奇怪。一对`try/catch`强制抛出了一个错误，这个“error”的值正好是`2`，`catch(a)`分句的变量声明接收这个值。啊，头疼。



没错，`catch`分句内的这个变量具有块级作用域，所以它可以成为在ES6之前的环境对块作用域的替代方案。



“但是……”你会说，“……谁想写这么丑的代码啊！”是的，没有人愿意写像CoffeeScript转译的代码。这不是重点。



重点是，工具会将ES6的代码转译成能在ES6之前的环境中工作的代码。你只尽管放心地使用块作用域，享受它带来的好处，构建时通过工具去产生能在部署时工作的代码。



这才是迁移ES6所有（唔，应该是大多数）特性的正确姿势：从ES6之前的环境过渡到ES6环境时，使用转码器生成能兼容ES5的代码。



## Traceur

google维护这一个叫做“Traceur”的项目，这个项目就是用来将ES6特性转译成兼容ES6之前（大多数是ES5）的环境的。TC39委员会用这个工具（和一些其他的工具）来测试他们指定的特性的语义。



你猜Traceur会把上面的那段代码转译成什么样呢？

```JavaScript
{
	try {
		throw undefined;
	} catch (a) {
		a = 2;
		console.log( a );
	}
}

console.log( a );
```

因此，通过使用这样的工具，我们可以使用块作用域时可以不用管运行环境到底是不是ES6了，因为`try/catch`从ES3开始就用这种方式工作了。



## Implicit vs. Explicit Blocks

在第三章，我们介绍了在使用块作用域时会在可维护性/可重构性上存在一些隐性的缺陷。是不是有什么方法可以让我们在使用块作用域时减少这种不利影响呢？



考虑以下这种`let`使用方式，它被称为“let块”或“let表达式”（对比之前的“let声明”）。

```JavaScript
let (a = 2) {
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

与隐式地劫持一个块不同，let表达式创建了一个显式的块并与其绑定。通过语法上手段强制把所有声明都放到块的顶部，来让代码更清晰。显式的块不仅更加显眼，而且更方便重构。这让我们更容易地判断块以及变量是属于哪个作用域的了。



作为一种样式，它借鉴了很多人在函数作用域中会使用的，将`var`声明手动移动/提升带函数顶部的这种做法。let表达式故意把声明放在块的顶部，所以，如果你不大面积地使用`let`，那么你的块作用域声明就会很容易辨认和维护。



但是依然还是存在一个小问题。ES6没有let表达式，官方的Traceur转译器也不接受这种形式的代码。



对此，我们有两种解决方案。使用ES6的合法语法，但是在代码规范上做一些妥协：

```JavaScript
/*let*/ { let a = 2;
	console.log( a );
}

console.log( a ); // ReferenceError
```

但是，工具就是用来解决问题的。所以，另一个办法就是，仍然显示地使用let表达式，用工具来帮我们将其转化为合法的、有效的代码。



因此，我写了一个“let-er”的工具来解决这个问题。“let-er”是构建时使用的代码转译器，它只会寻找let表达式，并转译它们。它不会管剩下的代码，包括let声明。你可以放心地把let-er作为ES6的第一个转译器，如果有需要的话，再把代码传送给像Traceur这样的工具。



另外，let-er还有一个配置项`--es6`，开启它（默认是关闭），会改变产生的代码的类型。开启它，let-er会生成ES6的代码，而不是ES3的`try/catch`的hack方案：

```JavaScript
{
	let a = 2;
	console.log( a );
}

console.log( a ); // ReferenceError
```

因此，你现在就可以在ES6之前的所有环境中使用let-er了，当你只关心ES6时，你只需要修改配置就可以产生ES6的代码了。



非常重要的一点：**你可以使用更好的、显示的let表达式形式**，即使它还不是官方标准。



## Performance

最后，我再说一下`try/catch`的性能，以及尝试回答下“为什么不用IIFE来创建作用域？”这个问题。



首先，`try/catch`的性能很差，但是没有合理的理由来说明它*必须*这么慢以及它*总是*这么慢。自从TC39官方支持在ES6转译器中使用`try/catch`，Traceur 团队已经向Chrome踢出改善`try/catch`性能的要求。



第二，IIFE不能和`try/catch`作为同类来比较，因为用函数包裹任意的代码会改变代码的意思，包括`this`,`return`,`break`和`continue`。IIFE不是一个具有普适意义的替代方案。它只能在特定情境下手动使用。



那么问题就变成了：你想用块作用域么？如果你想用，这些工具可以帮你实现。如果你不想用，你就继续使用`var`吧。



[Google Traceur](http://traceur-compiler.googlecode.com/git/demo/repl.html)

[let-er](https://github.com/getify/let-er)



