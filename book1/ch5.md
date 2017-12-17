在阅读这一章之前，我们希望你对作用域的工作原理有很好的理解。



我们要把我们的注意力放到这个语言的一个非常重要，但很难懂的一大概念：闭包。如果你已经看了之前关于词法作用域的那些内容，那么闭包就会很好理解。让我们来揭开它的神秘面纱吧！



如果你对词法作用域还有疑问，趁现在快去看下第二章吧。



### Enlightenment

对那些有点JavaScript经验但是从未真正理解过闭包的人来说，理解闭包可以看作是某种意义上的涅槃，需要为此付出巨大的努力和牺牲。



我想起当年的我，对JavaScript有扎实的了解，但是对闭包却一无所知。这个语言隐藏着另一个重要的方面，似乎只要掌握它，我就能得到巨大的进步。我曾经阅读了早期的框架源码，努力理解它是怎么工作的。我现在还能记得“模块模式”第一次闯进我的脑子时的恍然大悟的感觉。



我现在就要传授给你当时我无法理解，并因此花费数年才研究透的秘诀：**闭包在JavaScript中无处不在，你只需要认出它并拥抱它。**闭包不是插件工具，因此你不需要为此学习新的语法和模式。它也不是一件需要你像Luke一样需要接受原力训练才能使用和掌握的武器。



只要你的代码是基于词法作用域书写的，闭包就会发生。你甚至不必为了利用它们而特意创建闭包。闭包的创建和使用在你的代码中随处可见。你所缺少的是自主自愿地训练自己识别、接受和利用闭包的思维。



顿悟时应该是：**哦，原来我的代码中早已到处都是闭包了，我终于能理解他们了**。理解闭包就像Neo第一次看到Matrix时一样。



### Nitty Gritty

好了，夸张和无耻的电影引用够多了。



现在来一个直截了当的定义来让你理解和识别闭包：

> 闭包就是一个方法即使运行在它的词法作用域的外部也能够记住和访问它的词法作用域。



考虑以下代码：

```JavaScript
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

这段代码和我们讨论作用域嵌套时举例的代码很像。方法`bar()`能在外部的作用域中获取到变量`a`的值，因为词法作用域的引用规则（在这里，是一个RHS查询）。



这个是闭包么？



唔，技术上来说…可能是。但是根据我们上面给出的规则来说…不完全是。我认为，词法作用域的引用规则能最准确地解释`bar()`是怎么引用`a`的，但这些规则只是闭包的一部分，当然是很重要的一部分。



从纯学术的角度来看，就像上一段落所说的一样，方法`bar()`拥有一个涵盖`foo()`作用域的闭包（甚至说，涵盖了它能访问到的其他作用域，在这里就是全局作用域）。换句话说，`bar()`被关在了`foo()`里面。为什么？因为`bar()`嵌套在`foo()`里面。理由就是这么简单。



但是，这样定义的闭包无法直观地观察到，也无法让我们在上述代码片段中看到闭包是如何工作的。我们能清晰地看到词法作用域，但是闭包仍神秘地藏在代码之下。



考虑以下代码，它充分地展现了闭包：

```JavaScript
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, closure was just observed, man.
```



方法`bar()`的词法作用域可以访问`foo()`内部的作用域。然后我们又把`bar()`这个方法本身作为值来传递。在这个例子中，我们把`bar`引用的方法对象本身用作返回值。



运行了`foo()`之后，我们把它所返回的值（它内部的`bar()`方法）赋给了叫做`baz`的变量。然后我们调用`baz()`，实质上当然就是调用内部的`bar()`方法了，只是换了一个标识符引用它而已。



`bar()`就这样运行了。但是在这个例子中，它运行在了它所声明的词法作用域之外。



`foo()`运行之后，一般我们会认为`foo()`内部的整个作用域都会被销毁，因为我们知道引擎会使用垃圾回收机制把不再使用的内存释放掉。由于`foo()`的内容看起来是不会再使用到了，所以会很自然地想到去回收它们。



但是神奇的闭包不会让这件事发生。实际上，内部的作用域依然处于被使用的状态，因此也就不会被回收。谁在用它？**`bar()`方法它自己**。



由于`bar()`在`foo()`内部被声明，因此它的词法作用域涵盖了`foo()`的内部作用域。所以，`foo()`内部的作用域就会为了让`bar()`能在之后的任何时候都能引用而一直存活。



**`bar()`对这个作用域依然持有引用，这个引用就叫做闭包。**



因此，几微秒之后，当变量`baz`被调用时（也就是调用我们最初在方法内部标记的`bar`），它自然能访问代码书写阶段的词法作用域，因此它能够获取到变量`a`。



方法在它被书写是定义的词法作用域外被调用了。**闭包**让方法能够继续访问书写阶段所定义的词法作用域。



当然，任何能把方法作为值来传递的做法，并在其他地方调用这些方法的，都可以作为观察闭包的例子。

```JavaScript
function foo() {
	var a = 2;

	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}

function bar(fn) {
	fn(); // look ma, I saw closure!
}
```

我们把内部的方法`baz`传给了`bar`，并调用了这个方法（现在叫做`fn`）。此时通过获取`a`，它涵盖`foo()`的内部作用域的闭包就可以被观察到了。

传递函数也可以是间接的。

```JavaScript
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // assign `baz` to global variable
}

function bar() {
	fn(); // look ma, I saw closure!
}

foo();

bar(); // 2
```

无论我们是怎么把一个内部的方法传送到它的词法作用域之外的，它都会持有它最初的词法作用域的引用。无论我们何处运行它，闭包都会发挥作用。



### Now I Can See

之前的代码片段有点学术和做作地解释怎么使用闭包。但是我保证闭包不仅仅是一件很酷的玩具。我保证闭包在的代码中是无处不在的。现在让我们来搞懂这个事实。

```JavaScript
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Hello, closure!" );
```

我们把一个内部方法（叫做`timer`）传给了`setTimeOut(..)`。`timer`有一个涵盖`wait(..)`的闭包，甚至，保持并使用了对变量`message`的引用。



在调用`wait(..)`的一秒钟之后，它的内部作用域本应该早就被销毁了。但内部方法`timer`依然对这个作用域持有闭包所以，它不会被销毁。



在引擎的内部，内置的`setTimeout(..)`方法对参数持有引用，这些参数可能叫做`fn`，可能叫做`func`，也可能是其他类似的。引擎调用那个方法，也就是调用我们的内部方法`timer`，在此期间词法作用域一直保持完整。

**Closure**

或者，如果你熟悉jQuery（或其他JS框架）：

```JavaScript
function setupBot(name,selector) {
	$( selector ).click( function activator(){
		console.log( "Activating: " + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

我不知道你写的是哪种代码，但是我经常写代码，负责控制一个全封闭的全球无人飞行器，所以，这是完全可实现的。



玩笑归玩笑，本质上，无论何时何地，只要你将方法（它们可以访问到自己的词法作用域）作为第一级的值来传递，你就能看到这些方法在运用闭包。在那些定时器、事件处理器、ajax请求、跨窗口通信、web worker或其他的异步（同步）任务中，当你传入回调方法时，就准备好使用闭包吧！



**注意：**第三章介绍了IIFE模式。将IIFE作为一个观察闭包的例子，我是有点不同意的。

```JavaScript
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

虽然这段代码可以正常运行，但是它不是非常严格的可观察到闭包的代码。为什么？因为这个方法（这里我们管它叫做IIFE）不是运行在它的词法作用域之外的。它被调用的作用域和它被声明时所在的作用域是同一个作用域（外部/全局作用域也持有`a`）。`a`是通过正常的词法作用域查询所找到的，而不是通过闭包。



虽然从技术上来说，闭包是在声明时就产生的，但是它不是很容易就观察到的，就像俗话说的，一棵树倒在森林里，却没有人听到动静。



尽管IIFE不是一个闭包的例子，但是它确实创造了一个作用域，因此它也是我们最常用来创建一个封闭的作用域的方法。因此IIFE实际上和闭包的联系是非常紧密的，即使它们自身并没诶呦使用闭包。



亲爱的读者，现在请把书放下，我有一个任务要给你。打开你最近写的JavaScript代码，找找其中将方法作为变量和标识符的地方，在这些地方你已经在使用闭包了，即使你当时还不知道。



我等你。



好了，现在你懂了吧！



### Loops + Closure

用来说明闭包最常见的例子就是for循环。

```JavaScript
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**注意：**很多不理解闭包的开发者经常把方法放入循环，这会让代码格式检查器经常报错。我在这里给出如何正确地使用闭包，并发挥它的威力。但是检查器并没有那么聪明，它会假设你并不知道自己在干嘛，所以不管怎样，它都要发出警告。



我们本想通过运行上述代码来一秒一个地打印数字“1”、“2”…“5”。



但事实上，如果你运行了这段代码，你会看到“6”被打印了5次，每次相隔1秒。



嗯哼，咋回事捏？



首先，先来解释下`6`从哪里来的。for循环结束的条件是`i`不再`<=5`。那么当`i`等于6时，就可以结束了。因此结果反映的就是`i`结束循环时的值。



再多看一眼代码应该就能看出来了。timeout方法的回调都在循环结束后才运行的。事实上，即使每次循环里都是`setTimeout(.., 0)` ，所有的回调依然是在循环结束后才运行。因此每次打印的都是`6`。



引出一个更深入的问题。我们的代码缺了什么导致它的表现和语义上的期望的不一样。



缺少的就是我们想要让每个循环捕获当前循环的`i`的副本。但是按照作用域的工作方式，尽管这5个方法分别在不同的循环里被定义，**它们都被封闭在一个共享的全局作用域中**，所以事实上就只有一个`i`。



这样说的话，所有的方法自然是共享了同一个`i`的引用。循环结构可能会迷惑我们，让我们误以为还有其他更复杂的机制在工作。其实根本没有。一个接一个地声明5个timeout的回调和通过循环声明没有任何区别。



好了，回到正题。到底是缺了什么？我们需要更多的闭包作用域。具体地说，每一个循环都需要一个新的闭包作用域。



我们在第三章习得：IIFE通过声明一个方法并立即运行它来创建一个作用域。



考虑：

```JavaScript
for (var i=1; i<=5; i++) {
	(function(){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})();
}
```

这样写会起作用么？试试看。我等你。



好了，就不卖关子了。这样做是不行的。为什么？现在很明显有了更多的作用域。每个timeout方法的回调确实是被包裹在通过IIFE创建的它自己所属的迭代作用域中的。



如果作用域是空的话，仅仅将其包裹起来是没用的。请仔细看一下。我们的IIFE只是一个空的，啥事都没做的作用域。它需要一些别的东西才能对我们有用。



它需要自己的变量：每次迭代中的`i`的副本。

```JavaScript
for (var i=1; i<=5; i++) {
	(function(){
		var j = i;
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})();
}
```

哇哦！它起作用了！

其他人可能会另一种写法：

```JavaScript
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})( i );
}
```

当然，由于这些IIFE是函数，所以我们可以把传进去的`i`叫做`j`，也可以依然把他叫做`i`。无论如何，这段代码都起效了。



在迭代内使用的IIFE在每次迭代内部都创建了一个新的作用域，这使得timeout方法的回调能够每个迭代封闭起来。而每个迭代都有一个可供我们获取的正确的迭代值。



问题解决了！



### Block Scoping Revisited

仔细看我们对之前那个解决方法的分析。我们用IIFE在每个迭代里创建了一个新的作用域。换句话说，我们真正需要的是每个迭代的作用域块。第3章告诉我们`let`声明会劫持一个块并在其中声明一个变量。



**这实质上是将一个块转变成一个可封闭的作用域了。**因此，下面这段让人惊叹的代码也能起效：

```JavaScript
for (var i=1; i<=5; i++) {
	let j = i; // yay, block-scope for closure!
	setTimeout( function timer(){
		console.log( j );
	}, j*1000 );
}
```

不过，还没有结束哦！在for循环中`let`声明会有特殊表现。这个行为就是，变量不是在循环中只声明一次，而是**每次迭代**都会声明。并且， 随后的每次迭代中都会以上一次迭代结束时的值来初始化这个变量。

```JavaScript
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

酷不酷？块作用域和闭包联手就可以解决世界上所有的问题。我不知道你怎么样，反正这个特性能让我做一个快乐JavaScript程序员了。



## Modules

还有其他的一些代码模式可以增强闭包的作用，不过从表面看起来，它们有点像回调。让我们来看看它们中最强大的：模块。

```JavaScript
function foo() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}
}
```

正如代码显示的，并没有明显的闭包。只是有两个简单的私有数据变量`something`和`another`，和一对内部方法`doSomething()`、`doAnother()`。它们都有涵盖`foo()`内部作用域的词法作用域。



不过，现在考虑如下代码：

```JavaScript
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

这种模式在JavaScript中我们称之为模块。最常见的使用模块模式的方法是“模块暴露”。我们在这里展示的是它的变种。



让我们来仔细研究下这段代码。



首先，`CoolModule()`是一个方法，但是需要通过调用它来创建一个模块实例。如果不在外部函数中运行它，内部作用域和闭包都不会发生。



第二，`CoolModule()`方法返回一个用对象字面量语法`{ key: value, ... }`表示的对象。这个返回的对象引用了内部方法，但是没有引用内部的数据变量。内部数据变量依然是隐藏的、私有的。可以认为这个对象返回值是**模块的公共API**。



这个对象返回的值最后赋给了外部的`foo`变量，因此我们才能获取到API，像这样：`foo.doSomething()`。



**注意：**模块不一定要返回一个实在的对象（字面量），也可以直接返回一个内部方法。jQuery就是一个很好的例子。`jQuery`和`$`标识符是jQuery模块的公开API，但是它们本身只是一个方法（该方法有它自己的属性，因为万物皆对象）。



`doSomething()`和`doAnother()`方法拥有涵盖这个模块实例内部作用域的闭包（通过调用`CoolModule()`）。当我们通过引用模块返回的对象，把这些方法传递到词法作用域外部时，我们就建立了一个能让闭包被观察到并发挥它的作用的环境。



简单地总结，模块模式发挥作用有两个必要条件：

1.必须要有一个外部的封闭方法，并且它必须至少被调用一次（每次创建一个模块实例）。

2.这个封闭方法必须返回至少一个内部方法，因此它的内部方法就会有涵盖私有作用域的闭包，并能获取和修改私有状态。



只有一个成员方法的对象不是模块。从可观察的角度来说，方法调用后返回的对象如果只有成员变量没有成员方法的话，也不是一个模块。



上面的那段代码展示了独立的模块创建者`CoolModule()`，它可以被调用无数次，每次都会创建一个新的模块实例。当你只需要一个实例的时候，可以对代码进行一些变形，实现单例：

```javascript
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

这段代码中，我们把模块方法放进IIFE（详见第3章）中，立刻调用了模块方法并将它的返回值直接赋给单例模块实例标识符`foo`。



模块就是方法，因此他们可以接受传参：

```JavaScript
function CoolModule(id) {
	function identify() {
		console.log( id );
	}

	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```



另一个细小但很有用的变形：给作为公开API的返回对象命名。如下：

```JavaScript
var foo = (function CoolModule(id) {
	function change() {
		// modifying the public API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

通过在模块实例内部保持公开API对象的内部引用，你就可以**从内部**修改模块实例了，包括添加、移除方法和属性，以及改变它们的值。



### Modern Modules

大部分模块依赖加载器/管理器本质上是把模块定义封装进一个友好的API中。与其具体研究阐述某个具体的库，我觉得不如简单地阐述下概念：

```JavaScript
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

这段代码的关键部分是`modules[name] = impl.apply(impl, deps)`。调用了模块定义的封装函数（可传入任意依赖项），并将返回值，也就是模块的API，存入按名字跟踪的模块的内部列表中。



我会这么用它来定义模块：

```JavaScript
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

“foo”模块和“bar”模块都用一个返回值是公开API的方法定义。“foo”方法甚至还接收了“bar”的实例作为一个依赖参数，因此也能使用它。



花费一些时间来研究这些代码片段可以让我们能更好地运用闭包。需要明白的关键的一点是：模块管理器并没有什么“魔法”。它们完成了我之前指出的模块样式的两个特征：调用函数定义封装器、保留它的返回值作为模块的API。



换句话说，模块就只是模块，不管你是不是把它放到一个友好的封装工具里面。



### Future Modules

ES6为模块添加了一级语法支持。当通过模块系统加载时，ES6会将一个文件当做一个独立的模块。每个模块既可以导入其他的模块和具体的API，也可以导出它们自己的公开API。



**注意：**基于函数的模块不能稳定地被静态识别的样式（编译器无法识别）。因此他们的API语义在运行时才会被考虑。也就是说，你可以在运行时修改模块的API（看之前关于`publicAPI`的讨论）。



相比之下，ES6的模块API更加稳定（API不会在运行时改变）。由于编译器知道这一点，所以就会在编译阶段（文件加载时）检查一个指向一个被导入的模块的API成员是否真的存在。如果API引用不存在，那么编译器会在编译时就抛出异常，而不需要像传统的做法那样要等到运行时动态解析。



ES6模块没有“内联”格式，它们必须在独立的文件中定义（一个模文件一个模块）。浏览器/引擎有一个缺省的模块加载器（缺省的模块是可覆写的，不过这已经超出我们的讨论范围了），它会在模块文件被引入时就同步加载它。



考虑：

**bar.js**

```JavaScript
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**

```JavaScript
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```JavaScript
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**注意：**需要创建两个独立的文件“foo.js“和”bar.js“，分别写入上两段代码。然后，正如第三段代码所示，你就可以加载/导入这两个模块，并使用它们了。



`import`从一个模块中导入API到当前的作用域，每一个API成员绑定一个变量（在这个例子中就是`hello`）。`module`导入整个模块API，并与一个变量绑定（在这里就是`foo`和`bar`）。`export`为当前模块导出一个标识符（变量、方法）作为公开API。如果必要，这些操作可以在模块定义时使用任意次数。



模块文件中的代码会被当做包含在作用域闭包一样处理，就像之前提到的函数闭包模块一样。



## Review (TL;DR)

闭包似乎像只有少部分的勇士才能到达的，藏在JavaScript里面的神秘世界一样。但它实际上是一个标准，是关于在代码书写时的词法作用域环境中，方法作为值是如何传递的一个真相。



**闭包就是：一个方法即使是在它的词法作用域外部，也可以记住并获取到它的词法作用域**。



如果我们不能认出闭包，且不能了解它的原理，那么闭包就会阻碍我们前进，例如关于循环的那段代码。但闭包却是非常强大的工具，能够以各种形式实现模块等样式。



模块需要有两个必要的关键特征：1）调用一个外部的封装函数，这可创建内部作用域。2）封装函数的返回值必须包括至少一个内部方法的引用，这就可以拥有一个涵盖封装函数内部作用域的闭包了。



现在，我们就可以看见存在于代码中的闭包了，并将其为我们所用了。