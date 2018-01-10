尽管这个标题没有详细说明`this`机制，但是ES6中有一个主题，用非常重要的方式将`this`和词法作用域联系在一起。接下来，我们将简单探究它。



ES6加入了一种很特别的语法形式用来函数声明：“箭头函数”。它看起来就像这样：

```JavaScript
var foo = a => {
	console.log( a );
};

foo( 2 ); // 2
```

相对于繁琐冗长的`function`关键词，所谓的“胖箭头”作为便捷的方式，经常被提及。



除了让你在声明时少敲一点代码，箭头函数还有更重要的意义。



以下代码会有一个问题：

```JavaScript
var obj = {
	id: "awesome",
	cool: function coolFn() {
		console.log( this.id );
	}
};

var id = "not awesome";

obj.cool(); // awesome

setTimeout( obj.cool, 100 ); // not awesome
```

问题就在于，`cool()`函数没有与`this`绑定。有很多方法来解决这个问题，其中常用的一个就是`var self = this;`。

修改之后大概会是这样：

```JavaScript
var obj = {
	count: 0,
	cool: function coolFn() {
		var self = this;

		if (self.count < 1) {
			setTimeout( function timer(){
				self.count++;
				console.log( "awesome?" );
			}, 100 );
		}
	}
};

obj.cool(); // awesome?
```

修改后并没有把代码变得复杂，`var self = this;`的解决方式无需你理解整个问题，只是合理地绑定`this`，就把问题归结到我们熟悉的词法作用域上了。`self`变成了一个标识符，通过词法作用域和闭包来解决问题，不需要关心`this`绑定的过程中发生了什么。



人类不喜欢冗长的写法，尤其是要他们经常这么写。因此ES6的一个初衷就是要减缓这种情况，包括修复某些习惯语法的问题，比如这个。



ES6的解决方案，箭头函数，引入了一个叫做“词法 this”的行为。

```javascript
var obj = {
	count: 0,
	cool: function coolFn() {
		if (this.count < 1) {
			setTimeout( () => { // arrow-function ftw?
				this.count++;
				console.log( "awesome?" );
			}, 100 );
		}
	}
};

obj.cool(); // awesome?
```

简单点解释就是，箭头函数在`this`绑定方面和常规的函数表现得完全不一样。它抛弃了所有的`this`绑定的常规规则，而是把当前的封闭的词法作用域给到`this`的值上。



因此，在这个代码片段中，箭头函数的`this`没有被意外地解绑，它就自然而然地将`this`绑定到`cool()`函数上（正如调用后的结果所示）。



尽管箭头函数让代码变得更简便，但是我认为，它将程序员常犯的一个错误给编纂成语法了，即把“this绑定”和“词法作用域”两套规则混淆了。



换句话说：如果只是为了将其与词法引用混合在一起，为什么要使用麻烦又冗长的`this`风格编码范式？在代码中使用一种风格是很自然的事，不要把两种风格混合在一起。



**注意： **箭头函数另一个不好的地方是：他们是匿名函数不是具名的。关于为什么推荐使用具名函数而不是匿名函数，请看第3章。



我认为更好地解决这个问题的办法是，正确地使用`this`机制。

```JavaScript
var obj = {
	count: 0,
	cool: function coolFn() {
		if (this.count < 1) {
			setTimeout( function timer(){
				this.count++; // `this` is safe because of `bind(..)`
				console.log( "more awesome" );
			}.bind( this ), 100 ); // look, `bind()`!
		}
	}
};

obj.cool(); // more awesome
```

无论你喜欢用词法this或者箭头函数，还是可靠的`bind()`，你都要记住，箭头函数不仅仅只是少写了个”function“而已。



我们需要知道和了解它们之间各有特点的行为差异，这样我们才能更好地使用它们。



现在我们已经完全理解词法作用域（和闭包）了，理解词法this就是小菜一碟了。



























