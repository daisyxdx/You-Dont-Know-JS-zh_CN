在第一章中，我们抛掉了对`this`各种误解，明白了`this`是每个函数调用时产生的绑定，且完全取决于调用的位置（也就是函数是怎么被调用的）。



## Call-site

为了理解`this`绑定，首先要理解调用位置：函数是在什么代码的什么地方被调用的（**不是它声明的地方**）。我们必须先知道调用的地方，才能知道`this`指向什么。



一般来说，寻找调用位置就是寻找“函数被调用的位置”，但不是每次都能很轻易地找到，因为某些编程模式会把真正的调用位置隐藏起来。



最重要的就是**调用栈**（到达当前执行位置时，所调用的所有函数所形成的栈）。调用位置，是当前正在执行的函数的被调用的地方。



让我们来具体看看什么叫调用栈和调用位置：

```JavaScript
function baz() {
    // call-stack is: `baz`
    // so, our call-site is in the global scope

    console.log( "baz" );
    bar(); // <-- call-site for `bar`
}

function bar() {
    // call-stack is: `baz` -> `bar`
    // so, our call-site is in `baz`

    console.log( "bar" );
    foo(); // <-- call-site for `foo`
}

function foo() {
    // call-stack is: `baz` -> `bar` -> `foo`
    // so, our call-site is in `bar`

    console.log( "foo" );
}

baz(); // <-- call-site for `baz`
```

请多关注怎么正确地（从调用栈中）找到调用位置，因为这和`this`绑定有关。



**注意： **你可以在脑海中将调用栈可视化为有序的函数调用链，就像我们在上面的代码中写的注释一样。但是这种方法非常麻烦又容易出错。查看调用栈的另一个办法是使用浏览器中的调试工具。在上述代码中，你可以在`foo()`函数的第一行代码上打一个断点，或者直接在第一行中插入`debugger;`语句。当你运行代码时，调试器会停在这个位置，并展示达到当前行时所调用的所有函数的一个列表，这就是你需要的调用栈。因此，当你想要判断`this`绑定时，使用开发者工具来获取调用栈，栈中的第二项就是你找的调用位置。



## Nothing But Rules

现在我们将注意力放到：在函数运行时，调用位置是如何决定`this`绑定的是什么。



你需要观察调用位置来确定当前使用的是4个规则中的哪一个。首先，我们先来了解这4个规则。然后，我们再来了解，当多个规则被使用的时候，它们的优先级是怎样的。



### 默认绑定

我们先来了解最常用的情景：独立的函数调用。可以把这条规则看做是，当其他规则都无法使用时会默认使用的规则。



考虑：

```JavaScript
function foo() {
	console.log( this.a );
}

var a = 2;

foo(); // 2
```

如果你还没意识到的话，请注意了，全局作用域里的变量声明也就是全局对象的同名属性，比如`var a = 2`。它们不是复制而来的，它们是同一个东西，只是从两个不同的角度来看它们而已。



第二，当`foo()`被调用的时候，`this.a`就是全局变量`a`。为什么？因为在这个情境下，函数调用时，`this`运用了*默认绑定规则*，因此`this`指向了全局对象。



为什么这里用了*默认绑定规则* ？我们找到调用位置来看看`foo()`是怎么被调用的。在代码中，一个直接的没有任何修饰的函数引用调用了`foo()`。我们没有发现任何其他的规则可适用，因此就应用了*默认绑定规则*。



如果用了严格模式，全局对象就无法使用*默认绑定* ，因此，`this`就是`undefined`。

```JavaScript
function foo() {
	"use strict";

	console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```



一个很重要的小细节：虽然全部的 `this`绑定规则都是基于调用位置的，但是只有在`foo()`内部不运行在严格模式下，全局对象才有资格使用*默认绑定规则* ；如果是 `foo()` 的调用位置使用了严格模式，*默认绑定规则* 依然适用。

```JavaScript
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```

**注意： **故意将严格模式和非严格模式混淆使用，会让你的代码很难懂。整个程序要么是严格的，要么是非严格的。然而，有时候，你引入的第三方库使用的模式和你的代码不一样，所以你得小心这些小的兼容问题。

### 

### 隐式绑定

另一个绑定规则的场景：调用位置有一个上下文对象，也可以称作被某个对象拥有或包含，不过这种说法有点误导人。



考虑：

```JavaScript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

obj.foo(); // 2
```

首先，注意`foo()`是怎么被声明以及之后怎么作为引用属性被添加到`obj`上的。`foo()`不管是一开始就声明在`obj`里的，还是在之后才作为引用属性被添加到`obj`上的（正如这段代码所示），它都被`obj`对象所拥有或所包含。



调用位置使用`obj` 这个上下文来**引用**这个函数，因此，你可以说，在函数被调用的时候，`obj`对象“拥有”或“包含”了**函数引用**。



无论这种模式叫做什么，在`foo()`被调用的时候，它的落脚点就是`obj`对象。当函数引用有上下文对象时，*隐式绑定规则* 告诉我们，这个函数调用时的`this`绑定的就是这个对象。



因为`obj`是`foo()`调用时的`this`，所以，`this.a`就是`obj.a`。



对象属性引用链的最后一级和调用位置有关。例：

```JavaScript
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### 隐式丢失

`this`绑定最常见的问题之一就是，隐式绑定的函数会第丢失它绑定的对象，这种时候，往往就会使用*默认绑定规则*，绑定到全局对象或者`undefined`，这就取决于是不是严格模式了。



考虑：

```JavaScript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // function reference/alias!

var a = "oops, global"; // `a` also property on global object

bar(); // "oops, global"
```

`bar` 看起来像是`obj.foo`的引用，但其实，它只是一个指向`foo`本身的引用。调用位置是`bar()`，它是一个普通的没有修饰的调用，因此使用*默认绑定规则* 。



一种更微妙，更常见，也更意想不到的情况是，传入回调函数：

```JavaScript
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` is just another reference to `foo`

	fn(); // <-- call-site!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

doFoo( obj.foo ); // "oops, global"
```

传参是隐式赋值，这里我们传入的是一个函数，也就是隐式的引用赋值，因此，这段代码的结果和上一段代码的结果一样。



如果你传入回调函数的方法不是你自己声明的，而是语言内置方法，会怎么样呢？没什么区别，结果是一样的。

```JavaScript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

setTimeout( obj.foo, 100 ); // "oops, global"
```



JavaScript 环境中内置的 setTimeout() 函数实现和下面的伪代码类似：

```JavaScript
function setTimeout(fn,delay) {
	// wait (somehow) for `delay` milliseconds
	fn(); // <-- call-site!
}
```



就像我们看到的那样，回调函数丢失`this`绑定是非常常见的。还有一种行为会改变`this` :那个我们传入回调的函数有意地修改`this`。在很多主流的JS库中，时间处理函数很喜欢把回调函数的`this`强制绑定到别的对象上，比如说触发事件的DOM元素。这样子的机制有时候很有用，有时候就让人很恼火。不幸的是，这些工具往往不会让你自己来选择是否使用这种机制。



无论哪种情况，`this`  的改变都是无法意料的。因为，你其实无法控制你的回调函数的引用会怎么运行，所以你就无法控制调用位置。之后我们会介绍如何通过固定`this`来解决这个问题。



### 显式绑定

使用隐式绑定，我们必须让这个对象包含一个指向那个函数的引用，并通过这个属性间接（隐式）引用函数来把`this`绑定到这个对象上。



但是，如果我们不想让对象包含函数引用，就能让函数调用的`this`绑定到特定的对象上，该怎么做？



JavaScript中的所有函数都有一个共同的特性（通过他们的`[[Prototype]]`，之后会详细介绍），可以用来解决这个问题。具体来说，就是函数都有`call(..)`和`apply(..)` 方法。不过，严格来说，JavaScript的宿主环境有时会提供一些非常特殊的函数（用一种特别的方式来放置它们），它们是没有这两个方法的。但这是少数。大部分主要的函数以及你声明的函数，都可以使用`call(..)`和`apply(..)` 。



这两个方法是怎么工作的呢？他们的第一个参数都是一个作为`this`用的对象，然后用这个`this`来调用函数。因为你可以直接规定`this` ，所以我们将其称为*显示绑定*。



考虑：

```JavaScript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

通过*显示绑定* `foo.call(..)` 来调用`foo` ，我们强制让它的`this` 指向`obj`。



如果你传入了一个简单的原始值（`string`类型、`boolean`类型或`number`类型）作为`this`绑定的对象，这个原始值就会转换成它的对象形式（`new String(..)` ， `new Boolean(..)` ，或`new Number(..)`）。这被称为“boxing”（封箱）。



**注意： **对于`this`绑定来说，`call(..)`和`apply(..)`是完全一样的。它们的区别体现它们的传参方面，不过，现在我们不用关心这个。



不幸的是，只有*显示绑定* 还无法解决之前提出来的绑定丢失的问题。



#### 硬绑定

不过*显示绑定*的一种变形模式可以解决这个问题。考虑：

```JavaScript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

var bar = function() {
	foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` hard binds `foo`'s `this` to `obj`
// so that it cannot be overriden
bar.call( window ); // 2
```

我们来看看这个变形模式是怎么工作的。我们创建了`bar`函数，并在其内部手动调用`foo.call(obj)`，这样就强制`foo`调用时的`this `绑定到`obj`。无论你之后怎么调用`bar`，它总会用`obj`调用`foo`。这种强制的显示绑定方式称之为*硬绑定*。



硬绑定的封装函数最典型的应用场景是，建立传参和返回值联系：

```JavaScript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = function() {
	return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```



另一种使用方法是创建一个可重复使用的辅助类：

```JavaScript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

// simple `bind` helper
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

var obj = {
	a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```



由于*硬绑定* 是一种常用的模式，因此ES5提供了一个内置方法：`Function.prototype.bind`，可以这样使用它： 

```JavaScript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

`bind(..)`返回一个硬编码的函数，它会用你设置的参数作为原来那个函数被调用时的`this`绑定的上下文。



**注意： **在ES6，`bind(..)`提供的硬绑定函数拥有来自于原始目标函数的`.name`属性。例：`bar = foo.bind(..)`会有值为`“bound foo”` 的`bar.name`属性，这是在堆栈跟踪中显示的函数调用名称。



#### API调用上下文

很多库的函数和很多JavaScript与主机环境中内置的函数，会提供一个可选的参数，一般被称为“上下文”，它让在不使用`bind(..)`的情况下也能确保你的回调函数的`this`绑定确定的对象上。



例：

```JavaScript
function foo(el) {
	console.log( el, this.id );
}

var obj = {
	id: "awesome"
};

// use `obj` as `this` for `foo(..)` calls
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```

这些函数的内部基本都是通过`call(..)`和`apply(..)`来使用*显示绑定* 的。



#### `new`绑定

在讲第4个也是最后一`this`绑定规则之前，我们需要先来搞清楚一个在JavaScript常见的关于函数和对象的误解。



在传统的面向类的语言中，“constructor”（构造函数）是类中的一个特殊的方法。当用`new`操作符实例化一个类时，这个类中的构造函数就会被调用。看起来就像这样：

```JavaScript
something = new MyClass(..);
```

JavaScript也有`new`操作符，使用new的方式和面向类的语言基本一样。因此很多开发者认为JavaScript的new的机制也和那些语言相似。然而，JS里面的`new`和其他面向类的语言完全不一样。



首先，让我们重新定义下JS里的“构造函数”。在	JS里，构造函数**只是** 在使用`new`操作符时会被调用的 **函数**。它们和类没有关系，它们也不是类的实例化。它们甚至不能说是一种特殊的函数类型。它们只是使用`new`操作符时被调用的普通函数而已。



例如，`Number(..)`函数就是一个构造函数，引用自ES5.1的说明：

>15.7.2Number 构造函数
>
>当Number作为new表达式的一部分被调用时，它就是构造函数：它会初始化新创建的对象。

因此，几乎所有函数，包括像`Number(..)`这样的内置函数，都可以用`new`来调用，这样就可以调用构造函数了。需要清楚很重要的一点：并不存在“构造器函数”，事实上这是函数的构造的调用。



当用`new`调用函数时，或者说，构造函数被调用时，就会自动执行下面的操作：

1.凭空创建（或者说，构造）了一个崭新的对象

2.新的被构造的对象是用`[[Prototype]]`连接的。

3.新的被构造的对象就是那个函数被调用时的`this`绑定。

4.除非函数返回它自己的对象，否则，用`new`调用的函数会自动返回新的构造的对象。



步骤1、3、4和我们现在的话题有关。我们暂时跳过步骤2，在第5章再回来看看。



考虑：

```JavaScript
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

通过在前面修饰`new`调用`foo(..)` ，我们构造了一个新的对象，并把这个新的对象作为`foo(..)`被调用时`this`绑定的对象。**`new`就是用来给函数调用时的`this`绑定对象的第4个方法。** 我们管这个叫new绑定。



## 优先级

现在我们已经知道了函数调用中的4个`this`绑定规则。首先你要先找到调用位置，然后再判断应该用那种规则。但是，如果那个调用位置上有多个规则可适用，该用哪个呢？因此，就必须给这些规则设置优先级，下面我们将讨论这个优先级顺序到底是怎样的。



*默认绑定*优先级是最低的。所以我们先把它放在一边。



*隐式绑定*和*显示绑定*哪个优先级更高？我们来测试下：

```JavaScript
function foo() {
	console.log( this.a );
}

var obj1 = {
	a: 2,
	foo: foo
};

var obj2 = {
	a: 3,
	foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

由上可见，*显示绑定*比*隐式绑定*的优先级更高，也即是说，你应该先看看是否符合*显示绑定*，之后再考虑是否符合*隐式绑定*。



现在我们再来看看*new绑定*在优先级顺序中处于何处。

```JavaScript
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```



ok，*new绑定*比*隐式绑定*的优先级更高。那你觉得*new绑定*和*显示绑定*比较会怎样呢？



**注意： ** `new`和`call/apply`不能同时使用，因此不允许出现`new foo.call(obj1)`这种代码。但我们依然可以通过*硬绑定*来测试这两个规则的优先级顺序。



在用代码来测试之前，我们先来回顾下*硬绑定*是怎么工作的：`Function.prototype.bind(..)`创建的封装函数通过硬编码让它忽略它自己的`this`（无论这个`this`指向谁）绑定的对象，而使用我们手动提供的对象。



这样看来，似乎`硬绑定`（*显示绑定*的其中一个形式）的优先级比*new绑定*更高。



让我们来验证下：

```JavaScript
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

`bar`被硬绑定到了`obj1`上了，但是`new bar(3)`没有让`obj1.a`变成`3`。硬绑定得到的`bar(..)`能被`new`覆写，因此通过`new`，我们得到了它返回的对象并将其命名为`baz`，且`baz.a`的值为`3`。



回顾一下我们之前了解的“裸”绑定辅助类，就会很有意思：

```JavaScript
function bind(fn, obj) {
	return function() {
		fn.apply( obj, arguments );
	};
}
```

当你了解了这个辅助类是怎么工作的之后，你就会发现，根本无法给`obj`使用`new`操作符。



但是在ES5中内置的`Function.prototype.bine(..)`就更复杂了。下面是MDN提供的`bind(..)`的替代方案：

```JavaScript
if (!Function.prototype.bind) {
	Function.prototype.bind = function(oThis) {
		if (typeof this !== "function") {
			// closest thing possible to the ECMAScript 5
			// internal IsCallable function
			throw new TypeError( "Function.prototype.bind - what " +
				"is trying to be bound is not callable"
			);
		}

		var aArgs = Array.prototype.slice.call( arguments, 1 ),
			fToBind = this,
			fNOP = function(){},
			fBound = function(){
				return fToBind.apply(
					(
						this instanceof fNOP &&
						oThis ? this : oThis
					),
					aArgs.concat( Array.prototype.slice.call( arguments ) )
				);
			}
		;

		fNOP.prototype = this.prototype;
		fBound.prototype = new fNOP();

		return fBound;
	};
}
```

**注意：**上面的这种`bind(..)` 的替代方案和ES5内置的`bind(..)`产生的硬绑定的函数在面对`new`时的表现是不同的。因为替代方法不需要像内置方法一样通过`.prototype`来创建函数，而是通过其他方法间接地来实现相同的功能。当你使用这个替代方案时，请小心使用`new`。



下面这部分用到了`new `：

```JavaScript
this instanceof fNOP &&
oThis ? this : oThis

// ... and:

fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

在此我们不再深入研究这段代码（因为它很复杂，且我们目前的讨论无关），简单来说，这段代码会判断硬绑定函数之前是否被`new`修饰了（这会导致一个新的被构造的对象称为它的`this`），如果是，`this`就会指向新创建的对象而不是硬绑定提供的对象。



为什么`new`可以覆写*硬绑定* 呢？



主要原因是为了创建一个能够忽略`this`硬绑定、可预置传参的函数（可用`new`来构建对象）。`bind(..)`的另一个功能是：传入参数的除了第一个用来`this`绑定的以外，都会传给下层的函数（这种技术被称为“部分应用”，是“柯里化”的一种）。



例：

```JavaScript
function foo(p1,p2) {
	this.val = p1 + p2;
}

// using `null` here because we don't care about
// the `this` hard-binding in this scenario, and
// it will be overridden by the `new` call anyway!
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```



### 判断 `this`

现在，我们可以来总结下在函数调用位置决定`this`绑定对象的规则的优先级了。依次思考这些问题，在哪个问题停住，就使用哪个规则。

1.函数是否用`new`调用（**new 绑定**）？是的话，`this`就是新构建的对象。

`var bar = new foo()`

2.函数是否用`call`或`apply`调用的（**显示绑定**），或者是否用`bind`*硬绑定*？是的话，`this`就是显示指定的那个对象。

`var bar =foo.call(obj2)`

3.函数是否用一个上下文调用的（**隐式绑定**），或被一个对象所拥有？是的话，`this`就是那个上下文对象。

`var bar = obj1.foo()`

4.上述都不是的话， 就是**默认绑定**。如果是在严格模式，就是`undefined`；否则就是`global`对象。

`var bar = foo()`



就这么多。对于常规的函数调用，了解这些就足够明白`this`绑定了。em……不过还有些例外。



## 绑定例外

凡事总有例外，绑定规则也是。



在某些场景下，`this`绑定行为会出人意料，当你认为会用其他绑定规则时，实际上居然使用的是默认绑定（可以看之前的例子）。



### 忽略 `this`

如果你在`call`、`apply`和`bind`中传入`null `或`undefined`作为`this`绑定的入参时，这些值会被忽略，在这种情境下，就会使用*默认绑定*。

```JavaScript
function foo() {
	console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```

什么时候你可能会传入`null`来作为`this`的绑定呢？



使用`apply(..)`把数组的值分离出来作为参数传入函数是很常见的。相似的，`bind(..)`可以对参数进行柯里化（预置值）。这是非常有用的。

```JavaScript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// spreading out array as parameters
foo.apply( null, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

这两种方式都需要传入一个`this`绑定的对象作为第一个参数。像这里，你根本不关心`this`绑定的是什么，但同时你又需要一个缺省的值，`null`就是一个不错的选择。



**注意： **虽然在本书中，我们不涉及，但是在ES6中有展开操作符：`...`，它可以不需要`apply(..)`，直接把一个数组展开成一组参数，传入函数，如：`foo(..[1,2])`就相当于`foo(1,2)` ，这样就可以避免不需要的`this`绑定。可惜的是，ES6中没有柯里化的相关语法，所以`bind(..)`还是需要传入作为`this`绑定对象的参数。



然而，总是用`null`来作为`this`的绑定存在一定的隐患。如果在某个函数调用时使用`null`作为`this`绑定的对象（例如，你无法控制的某个第三方库函数），且这个函数调用了`this`，那根据*默认绑定规则*，就会引用甚至意外修改`global`对象（在浏览器环境下就是`window`）。



很明显，这种方式可能会导致许多难以分析和追踪的bug。



#### 更安全的 `this`

一种“更安全”的做法是传入一个把`this`绑定到它身上也不会对程序产生任何副作用的特殊的对象。借鉴网络（还有军队），我们可以创建一个“DMZ”（de-militarized zone非军事地带）对象：就是一个空的非委托对象（关于委托请看第5章和第6章）。



当我们不关心`this`绑定的对象是什么时，我们只要传入DMZ对象，就能保证`this`指向这个空对象，不会影响到`global`对象。



由于这是个空对象，因此我个人比较愿意将它命名为`ø`（这是数学中表示空集的小写符号）。在Mac电脑上，`⌥`+`o` (option+`o`)就可以打出这个符号了。如果你不喜欢用这个符号，你想管它叫啥都可以。



无论你管它叫什么，创建一个**完全空的对象**最简单的方法就是`Object.create(null)`（详见第5章）。`Object.create(null)`和`{}`类似，但它不需要`Object.prototype`，所以它比`{}`更空。

```JavaScript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// our DMZ empty object
var ø = Object.create( null );

// spreading out array as parameters
foo.apply( ø, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

使用变量名 ø 不仅让函数变得更加“安全”，而且可以提高代码的可读性，因为 ø 表示“我希望`this`是空”，这比`null`的含义更清楚。不过再说一遍，你可以用任何喜欢的名字来命名 DMZ 对象。



### 间接引用

另一件需要注意的事是，你可以（有意或无意地）创建函数的“间接引用”。在这种情况下，当函数引用被调用时，就会应用*默认绑定规则*。



发生*间接引用*最常见的情况是赋值：

```JavaScript
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```

表达式`p.foo=o.foo`的*返回值*是目标函数的引用。因此，调用位置是`foo()`，不是`p.foo()`或`o.foo()`。所以这里用的是*默认绑定*。



记住：无论程序是在何种情况下应用了*默认绑定规则*，决定`this`绑定对象的，不是调用位置是否处于严格模式下，而是调用的函数体本身是否处于严格模式下：非严格模式下是`global`，严格模式是`undefined`。



### 软绑定

我们已经知道了*硬绑定*通过强制绑定一个特定的`this`对象（除非你用`new`覆盖它），防止函数调用意外地变成*默认绑定*。但是*硬绑定*使得函数缺乏了灵活性，因为它使得之后无法用*隐式绑定* 或 *显示绑定*覆写`this`了。



我们可以构造一个*软绑定*工具类来实现我们想要的效果：

```JavaScript
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this,
			curried = [].slice.call( arguments, 1 ),
			bound = function bound() {
				return fn.apply(
					(!this ||
						(typeof window !== "undefined" &&
							this === window) ||
						(typeof global !== "undefined" &&
							this === global)
					) ? obj : this,
					curried.concat.apply( curried, arguments )
				);
			};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}
```

除了*软绑定*外，上面的这个`softBind(..)`工具的原理和ES5内置的`bind(..)`是类似。这个方法做了这么一件事情：在调用位置检查`this`：如果`this`是`global`或者是`undefined`，`this`就指向之前指定的对象（`obj`）；反之，`this`就原封不动。这个工具类还可以支持柯里化（详见之前讨论的`bind`）。



实际应用下这个软绑定：

```JavaScript
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   <---- look!!!

fooOBJ.call( obj3 ); // name: obj3   <---- look!

setTimeout( obj2.foo, 10 ); // name: obj   <---- falls back to soft-binding
```

软绑定的`foo()`函数可以手动地将`this`绑定到`obj2`和`obj3`上，但是当用到*默认绑定*时，就会回到`obj`上。



## 词法 `this`

一般的函数遵守以上4种规则。但是ES6引入的箭头函数不遵守。



箭头函数不是使用`function`关键字定义的，是用“胖箭头”`=>`定义的。箭头函数的`this`是根据外层（函数或全局）作用域决定的。



我们来看看箭头函数的词法作用域：

```JavaScript
function foo() {
	// return an arrow function
	return (a) => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```

在`foo()`里的箭头函数会捕获`foo()`调用时的`this`。由于`foo()`的`this`绑定到`obj1`，`bar`的`this`也绑定到`obj1`。箭头函数的绑定不能被覆盖（即使用`new`也不能）。



最常见的使用场景就是回调，比如事件处理器和计时器：

```JavaScript
function foo() {
	setTimeout(() => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

箭头函数的`this`绑定功能可以替代`bind(..)`，它看起来很棒，可是你得注意，箭头函数本质上是抛弃了传统的`this`机制，而传统的机制是有助于更好的理解词法作用域的。在es6之前，我们已经有了一种相对比较常见的模式来做这件事，这种模式和es6的箭头函数没什么区别。

```JavaScript
function foo() {
	var self = this; // lexical capture of `this`
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

当你不想使用`bind(..)`时，`self = this`和箭头函数看起来都是不错的办法，但它们本质上是在逃避`this`，而不是理解和拥抱它。



如果你经常使用`this`风格的代码，但绝大部分都是在用`self = this`或箭头函数这种技巧来避免`this`机制。那你或许应该：

1.只使用词法作用域，并完全抛弃错误的`this`风格代码。

2.完全拥抱`this`机制，包括在必要时使用`bind(..)`，尽量避免使用`self = this`和箭头函数这种”词法this“的技巧。



程序可以同时使用这两种风格的代码（词法和`this`），但是在同一段代码和同一段程序里混用两种机制会使代码难以维护，也更难编写。



## Review (TL;DR)

确定`this`绑定是什么，首先需要找到所在函数的调用位置。然后再从上到下地运用以下四种规则：

1.如果是用`new `调用的，那么就是新创建的构造对象。

2.如果使用`call`或`apply`（或`bind`），那么就是那个特定的对象。

3.如果是一个拥有它的上下文对象调用的，那么就是这个上下文对象。

4.默认：在严格模式就是`undefined`，在非严格模式就是全局对象。



一定要注意，有些调用可能在无意中使用了*默认绑定规则*。如果想更安全地忽略`this`绑定，你可以使用一个DMZ对象来避免全局对象受到污染，比如 `ø = Object.create(null)` 。



除了这4种标准的绑定规则，还有es6的箭头函数，它是根据当前的词法作用域来决定`this`的。也就是说，箭头函数会抛弃`this`在函数调用时绑定的对象。它本质上就是`self = this`的替代品。


​			
​		
​	











































