# JavaScript轻量级函数式编程
# 章节 7: 闭包 vs 对象

数年前，Anton Van Straaten 创造了一个非常有名且被常常引用的[禅理](https://www.merriam-webster.com/dictionary/koan) 来举例和证实一个闭包和对象之间重要的关系。

> 令人尊敬的大师 Qc Na 曾经和他的学生 Anton 一起散步。Anton 希望引导大师到一个讨论里，说到：大师，我曾听说对象是一个非常好的东西，是这样么？Qc Na 同情地看着他的学生回答到, "愚笨的弟子，对象只不过是可怜人的闭包"
>
> 被批评后，Anton离开他的导师会到了自己的住处，致力于学习闭包。他认真的阅读整个 ”匿名函数：终极…"系列论文和它的姐妹篇，并且实践了一个基于闭包系统的小的 Scheme 解析器。他学了很多，盼望展现给他导师他的进步。
>
> 当他下一次与 Qc Na 一同散步时，Anton 试着提醒他的导师，说到 “导师，我已经勤奋地学习了这件事，我现在明白了对象真的是可怜人的闭包” ，Qc Na 用棍子戳了戳 Anton 回应到，“你什么时候才能学会，闭包才是可怜人的对象”。在那一刻， Anton明白了什么。
>
> Anton van Straaten 6/4/2003
>
> http://people.csail.mit.edu/gregs/ll1-discuss-archive-html/msg03277.html

原帖尽管简短，却有更多的内容关于起源和动机，我强烈推荐你阅读帖子来调整你的观念来理解本章。

我观察到很多人读完这个会对其中的聪明智慧傻笑，却继续不改变他们的想法。但是，这个禅理（来自 Bhuddist Zen 观点）是促使读者进入其中对立真相的辩驳中。所以，回去再读一遍，现在再读一遍。

到底是哪个？是闭包是可怜人的对象，还是对象是可怜人的闭包？或都不是？或都是？这仅仅带出闭包和对象在某些方面是相同的？

还有它们中哪个与函数式编程相关？拉一把椅子过来并且仔细考虑一会儿。如果你愿意，这一章将是一个精彩的迂回之路，一个远足。

## 达成共识

先确定一点，当我们谈及闭包和对象我们都达成了共识。我们显然是在讨论 JavaScript 如何处理这两种机制的上下文中进行讨论的，并且特指的是讨论简单函数闭包（见第二章的”保持作用域“）和简单对象（键值对的集合）。

一个简单函数闭包：

```js
function outer() {
	var one = 1;
	var two = 2;

	return function inner(){
		return one + two;
	};
}

var three = outer();

three();			// 3
```

一个简单对象：

```js
var obj = {
	one: 1,
	two: 2
};

function three(outer) {
	return outer.one + outer.two;
}

three( obj );		// 3
```

但提到“闭包“时，很多人会想很多额外的事情，例如异步回调甚至是封装和信息隐藏的模块模式。同样，”对象“会让人想起类、`this`、原型和大量其它的工具和模式。

随着深入，我们会需要小心地处理部分额外的相关内容，但是现在，尽量只记住闭包和对象最简单的释义——这会减少很多探索过程中的迷惑。

## 相像

闭包和对象之间的关系可能不是那么明显。让我们先来探究它们之间的相似点。

为了给这次讨论一个基调，让我简述两件事：

1. 一个没有闭包的编程语言可以用对象来模拟闭包。
2. 一个没有对象的编程语言可以用闭包来模拟对象。

换句话说，我们可以认为闭包和对象是一样东西的两种表达方式。

### 状态

思考下面的代码：

```js
function outer() {
	var one = 1;
	var two = 2;

	return function inner(){
		return one + two;
	};
}

var obj = {
	one: 1,
	two: 2
};
```

`inner()`  和 `obj` 对象持有的作用域都包含了两个元素状态：值为 `1` 的 `one` 和值为 `2` 的 `two`。从语法和机制来说，这两种声明状态是不同的。但概念上，他们的确相当相似。

事实上，表达一个对象为闭包形式，或闭包为对象形式是相当简单的。接下来，尝试一下：

```js
var point = {
	x: 10,
	y: 12,
	z: 14
};
```

你是不是想起了一些相似的东西？

```js
function outer() {
	var x = 10;
	var y = 12;
	var z = 14;

	return function inner(){
		return [x,y,z];
	}
};

var point = outer();
```

**说明：**每次被调用时 `inner()` 方法创造并返回了一个新的数组（亦然是一个对象）。这是因为 JS 不提供我们返回多个数据却不包装在一个对象中的能力。这并不是严格意义上一个违反我们对象类似闭包的说明工作，因为这只是一个暴露／运输具体值的实现，我们可以声明地忽视这个临时中间对象通过另一种方式：`var [x,y,z] = point()`。从开发者工程学角度，值应该被单独存储并且通过闭包而不是对象追踪。

如果你有一个嵌套对象会怎么样？

```js
var person = {
	name: "Kyle Simpson",
	address: {
		street: "123 Easy St",
		city: "JS'ville",
		state: "ES"
	}
};
```

我们可以用嵌套闭包来表示相同的状态：

```js
function outer() {
	var name = "Kyle Simpson";
	return middle();

	// ********************

	function middle() {
		var street = "123 Easy St";
		var city = "JS'ville";
		var state = "ES";

		return function inner(){
			return [name,street,city,state];
		};
	}
}

var person = outer();
```

让我们尝试另一个方向，从闭包转为对象：

```js
function point(x1,y1) {
	return function distFromPoint(x2,y2){
		return Math.sqrt(
			Math.pow( x2 - x1, 2 ) +
			Math.pow( y2 - y1, 2 )
		);
	};
}

var pointDistance = point( 1, 1 );

pointDistance( 4, 5 );		// 5
```

`distFromPoint(..)` 封装了 `x1` 和 `y1` ，但是我们也可以通过传入一个具体的对象作为值：

```js
function pointDistance(point,x2,y2) {
	return Math.sqrt(
		Math.pow( x2 - point.x1, 2 ) +
		Math.pow( y2 - point.y1, 2 )
	);
};

pointDistance(
	{ x1: 1, y1: 1 },
	4,	// x2
	5	// y2
);
// 5
```

`point`对象状态明确地被传入替换了闭包隐式持有的状态。

#### 行为，也是一样

对象和闭包不仅是表达状态集合的方式，而且他们也可以包含函数或者方法。将数据和行为捆绑有一个充满想象力的名字：封装。

思考：

```js
function person(name,age) {
	return happyBirthday(){
		age++;
		console.log(
			"Happy " + age + "th Birthday, " + name + "!"
		);
	}
}

var birthdayBoy = person( "Kyle", 36 );

birthdayBoy();			// Happy 37th Birthday, Kyle!
```

内部函数 `happyBirthday()` 封闭了 `name` 和 `age` 所以内部的函数也持有了这个状态。 

我们也可以通过对象内部的 `this` 绑定来获取这个能力：

```js
var birthdayBoy = {
	name: "Kyle",
	age: 36,
	happyBirthday() {
		this.age++;
		console.log(
			"Happy " + this.age + "th Birthday, " + this.name + "!"
		);
	}
};

birthdayBoy.happyBirthday();
// Happy 37th Birthday, Kyle!
```

我们仍然通过 `happyBrithday()` 函数来表达对状态数据的封装，但是用对象代替了闭包。同时我们没有显式给函数传递一个对象（如同先前的例子）；JavaScript 的 `this` 绑定可以创造一个隐式的绑定。

从另一方面分析这种关系：闭包将单个函数与一系列状态结合起来,而对象却在保有相同状态的基础上,允许任意数量的函数来操作这些状态。

事实上，我们可以暴露一系列的方法在一个作为接口的闭包。思考一个包含了两个方法的传统对象：

```js
var person = {
	firstName: "Kyle",
	lastName: "Simpson",
	first() {
		return this.firstName;
	},
	last() {
		return this.lastName;
	}
}

person.first() + " " + person.last();
// Kyle Simpson
```

用闭包而不用对象，我们可以表达这个程序为：

```js
function createPerson(firstName,lastName) {
	return API;

	// ********************

	function API(methodName) {
		switch (methodName) {
			case "first":
				return first();
				break;
			case "last":
				return last();
				break;
		};
	}

	function first() {
		return firstName;
	}

	function last() {
		return lastName;
	}
}

var person = createPerson( "Kyle", "Simpson" );

person( "first" ) + " " + person( "last" );
// Kyle Simpson
```

尽管这些程序看起来感觉有点反人类， 但它们实际上只是相同程序的不同实现。

### (不)可变

许多人最初都认为闭包和对象行为的差别源于可变性；闭包会阻止来自外部的变化而对象则不然。但是，结果是，这两种形式都有典型的可变行为。

正如第 6 章讨论的，这是因为我们关心的是**值**的可变性，值可变是值本身的特性，不在于在哪里或者如何被赋值的。

```js
function outer() {
	var x = 1;
	var y = [2,3];

	return function inner(){
		return [ x, y[0], y[1] ];
	};
}

var xyPublic = {
	x: 1,
	y: [2,3]
};
```

在 `outer()` 中字面变量 `x` 存储的值是不可变的————记住，定义的基本类型如 `2` 是不可变的。但是 `y` 的引用值，一个数组，绝对是可变的。这点对于 `xyPublic` 中的 `x` 和 `y` 属性也是完全相同的。

通过指出 `y` 本身是个数组我们可以强调对象和闭包在可变这点上没有关系，因此我们需要将这个例子继续拆解：

```js
function outer() {
	var x = 1;
	return middle();

	// ********************

	function middle() {
		var y0 = 2;
		var y1 = 3;

		return function inner(){
			return [ x, y0, y1 ];
		};
	}
}

var xyPublic = {
	x: 1,
	y: {
		0: 2,
		1: 3
	}
};
```

如果你认为这个如同 "海龟 (又称, 对象) 背地球"，在最底层，所有的状态数据都是基本类型，而所有基本类型都是不可变值。

不论是用嵌套对象还是嵌套闭包代表状态，这些被持有的值都是不可变的。

### 同构

同构这个概念最近在 JavaScript 圈被经常提出，它通常被用来指代码可以同时被服务端和浏览器端使用／分享。 我不久以前写了一篇博文说明这种对同构这个词的使用是错误的，隐藏了它实际上确切和重要的意思。

这里我是博文部分的节选：

> 同构的意思是什么？当然，我们可以用数学词汇，社会学或者生物学讨论它。同构最普遍的概念是你有两个类似但是不相同的结构。
>
> 在这些所有的惯用法中，同构和相等的区别在这里：如果两个值在各方面完全一致那么它们相等，但是如果它们表现不一致却仍有一对一或者双向映射的关系那么它们是同构。
>
> 换而言之，两件事物A和B如果你能够映射（转化）A到B并且能够通过反向映射回到A那么它们就是同构。

回想第二章的简单数学回顾，我们讨论了函数的数学定义是一个输入和输出之间的映射。我们指出这在学术上称为态射。同构是双射（双向）态射的特殊案例，它需要不仅仅映射必须可以从任意一边完成，而且在任何形式行为完全相同。

 不去思考这些关于数字的，让我们关联同构到代码。再一次引用我的博文：

> 如果有的话同构 JS 是怎么样的？它可能是你有一集合的 JS 代码转化为了另一集合的 JS 代码，并且（重要地）如果你想你可以把转化后的转为之前的。
>

正如我们之前通过闭包如同对象和对象如同闭包为例声称的一样，这种可以通过两者中的任何一个方式交替展现的。就一点来说，它们互为同构。

简而言之，闭包和对象是状态的同构表示 （和它们相关的功能性）。

下次你听到谁说 ”X 与 Y 是同构的“，他们的意思是， ” X 和 Y 可以从两种中的任意一方转化到另一方， 并且无论怎样都保持了相同的特性。“

### 内部结构

所以，我们可以从我们写的代码角度想象对象是闭包的一种同构展示。但我们也可以观察到闭包系统可以被实现——最合适的就是——通过对象。

在这方面考虑一下：在如下的代码中， 在 `outer()` 已经运行后，JS 如何为了 `inner()` 保持引用来持续记录变量 `x`？

```js
function outer() {
	var x = 1;

	return function inner(){
		return x;
	};
}
```

我们会想到作用域，`outer()` 的所有变量定义通过一个变量的属性定义。因此，从概念上讲，在内存中的某个地方，是类似这样的。

```js
scopeOfOuter = {
	x: 1
};
```

接下来对于 `inner()` 函数，但创建时，它获得了一个叫做 `scopeOfInner` 的（空）作用域对象，这个对象被其 `[[Prototype]]` 连接到 `scopeOfOuter` 对象，近似这个：

```js
scopeOfInner = {};
Object.setPrototypeOf( scopeOfInner, scopeOfOuter );
```

接着，内部的 `inner()`, 当他建立词法变量 `x` 的引用时，实际更像这样：

```js
return scopeOfInner.x;
```

`scopeOfInner` 并没有一个 `x` 的属性，当他的 `[[Prototype]]` 连接到拥有 `x` 属性的 `scopeOfOuter`。访问 `scopeOfOuter.x` 通过原型委托返回值 `1` 作为结果。

这样，我们可以近似认为为什么 `outer()` 的作用域甚至在当它执行完都被保留（通过闭包）：因为 `scopeOfInner` 对象连接到 `scopeOfOuter` 对象，因此，保持了这个对象和它的属性存在的很好。

现在，这都只是概念。我没有从字面上说 JS 引擎使用对象和原型。但是它**可能**类似这样工作是完全合理的。

许多语言实际上通过对象实现了闭包。另一些语言用闭包的概念实现了对象。但我们让读者使用他们的想象力思考这是如何工作的。


## 同根异枝

所以闭包和对象是等价的，对吗？不完全是，我打赌它们比你在读本章前想的更加相似，但是它们仍有重要的区别点。

这些区别点不应当被视作缺点或者不利于使用的论点；这是错误的观点。它们应当被视作特性和优点使一个（闭包或对象）或另一个更适合做一个给定的任务。

### 结构可变性

从概念上讲，闭包的结构不是可变的。

换而言之，你永远不能从闭包添加或移除状态。闭包是一个表示对象在哪里声明的特性（被固定在编写／编译时间），并且不受任何条件的影响 — 当然假设你使用严格模式并且／或者避免使用作弊手段例如 `eval(..)`。

**注意：**JS引擎可以技术上过滤一个对象来清除其作用域中不再被使用的变量，但是这是一个对于开发者透明的先进优化。无论引擎是否实际做了这类优化，我认为对于开发者假设闭包是作用域优先而不是变量优先是最安全的。如果你不想保留它，就不要封闭它（在闭包里）！

但是，对象默认是完全可变的，你可以自由的添加或者移除 (`delete`) 一个对象的属性／索引，只要对象没有被冻结（`Object.freeze(..)`）

这或许是代码可以根据程序中运行时条件追踪更多（或更少）状态的优势。

举个例子，让我们思考追踪游戏中的按键事件。几乎可以肯定，你会考虑使用一个数组来做这件事：

```js
function trackEvent(evt,keypresses = []) {
	return keypresses.concat( evt );
}

var keypresses = trackEvent( newEvent1 );

keypresses = trackEvent( newEvent2, keypresses );
```

**注意**：你能否认出为什么我使用 `concat(..)` 而不是直接 `push(..)` 向 `keypresses` 数组？因为在函数式编程中，我们通常希望对待数组如同不可变数据结构，可以被创建和添加，但不能直接改变。我们剔除了显式重新赋值带来的邪恶副作用（更多的稍后再谈）

尽管我们不在改变数组的结构，但当我们希望时我们也可以。稍后详细介绍。数组不是仅有的方式来记录这个 `evt` 对象的增长“列表”。我们可以使用闭包：

```js
function trackEvent(evt,keypresses = () => []) {
	return function newKeypresses() {
		return [ ...keypresses(), evt ];
	};
}

var keypresses = trackEvent( newEvent1 );

keypresses = trackEvent( newEvent2, keypresses );
```

你看出这里发生了什么？

每次我们添加一个新的事件到这个“列表”，我们创建了一个包装了现有 `keypresses()` 方法（闭包）的新闭包，这个新闭包捕获了当前的 `evt` 。当我们调用 `keypresses()` 函数，它将成功地调用所有的内部方法，创建一个包含所有独立封装的 `evt` 对象的中间数组。再次说明，闭包是一个追踪所有状态的机制；这个你看到的数组只是一个对于需要一个方法来返回函数中多个值的具体实现。

所以哪一个更适合我们的任务？毫无意外，数组方法可能更合适一些。闭包的不可变结构意味着我们的唯一选项是封装跟多的闭包在里面。对象默认是可扩展的，所以我们需要增长这个数组就足够了。

顺便一提，尽管我们表现出结构不可变或可变是一个闭包和对象之间的明显区别，然而我们使用对象作为一个不可变数据的方法实际上使之更相似而非不同。

数组每次添加就创造一个新数组（通过 `concat(..)`）就是把数组对待为结构不可变，这个概念上对等于通过适当的设计使闭包结构上不可变。

### 私有

当对比分析闭包和对象时可能你思考的第一个区分点就是闭包通过词法作用域提供“私有”状态，而对象将一切做为公共属性暴露。这种私有有一个精致的名字：信息隐藏。

考虑词法闭包隐藏：

```js
function outer() {
	var x = 1;

	return function inner(){
		return x;
	};
}

var xHidden = outer();

xHidden();			// 1
```

现在同样的状态公开：

```js
var xPublic = {
	x: 1
};

xPublic.x;			// 1
```

这里有一些明显的区别在普遍的软件工程原理——考虑下抽象，这种模块模式有着公开和私有APIs等等。但是让我们约束我们的讨论在函数式编程的角度，毕竟，这是一本关于函数式编程的书！

#### 可见性

可能看起来对于状态追踪隐藏信息的能力是一个被渴望的特性，但是我认为函数式编程者可能持反对观点。

在一个对象中管理状态为公开属性的一个优点是这使你状态中的所有数据更容易枚举（迭代）。思考下你想访问每一个按键事件（从之前的那个例子）并且存储到一个数据库，使用一个这样的工具：

```js
function recordKeypress(keypressEvt) {
	// database utility
	DB.store( "keypress-events", keypressEvt );
}
```

If you already have an array -- just an object with public numerically-named properties -- this is very straightforward using a built-in JS array utility `forEach(..)`:

如果你已经有一个数组，正好是一个拥有公开的用数字命名属性的对象——非常直接地使用 JS 对象的内建工具 `forEach(..)`：

```js
keypresses.forEach( recordKeypress );
```

但是，如果按键列表被隐藏在一个闭包里，你不得不在闭包内暴露一个享有特权访问数据的公开 API 工具。

举例而说，我可以给我们的闭包——keypresses 例子自己的 forEach，像数组内建有的：

```js
function trackEvent(
	evt,
	keypresses = {
		list() { return []; },
		forEach() {}
	}
) {
	return {
		list() {
			return [ ...keypresses.list(), evt ];
		},
		forEach(fn) {
			keypresses.forEach( fn );
			fn( evt );
		}
	};
}

// ..

keypresses.list();		// [ evt, evt, .. ]

keypresses.forEach( recordKeypress );
```

对象状态数据的可见性让我们能更直接地使用它，而闭包遮掩状态让我们更艰难地处理它。

#### Change Control

#### 变更控制

如果词法变量被隐藏在一个闭包中，只有闭包内部的代码才能自由的重新赋值，在外部修改 x 是不可能的。

正如我们在第6章看到的，提升代码可读性的唯一真相就是减少表面掩盖，读者必须可以预见到每一个给定变量的行为。

词法（作用域）重新赋值的局部就近是一个为什么我找不到 const 特性有用的很大原因。作用域（闭包如此）通常应该尽可能小，这意味着重新赋值只会影响几行代码。在上面的 `outer() `中，我们可以快速地检查没有一行代码重设了 x，所有意图和目的表现像一个常量。

这类保证对于例如我们对函数纯净的信任是一个重要的贡献。

换而言之，xPublic.x 是一个公共属性，程序的任何部分都拥有 xPublic 的一份引用，有了默认重设 xPublic.x 到别的值的能力。这会让很多行代码需要被考虑。

这是为什么在第 6 章，我们视 Object.freeze(..) 为一个快速而凌乱的方案用使所有的对象属性只读（可写性：非）的方式，让它们不能被不可预测的重设。

不幸的是，`Object.freeze(..)` 是极端且不可逆的。

使用闭包，我们可以有一些代码有权限改变，而剩余的程序是受限的。当我们冻结一个对象，代码中没有任何部分可以被重设。此外，一旦一个对象被冻结，它不能被解冻，所以所有属性在程序运行期间都保持只读。

在我想允许重新赋值但是在表层限制的地方，闭包比起对象更方便和灵活。在我不想重新赋值的地方，一个冻结的对象比起重复 const 声明在我所有的函数中更方便一些。

许多函数式编程者采取了一个强硬的立场在重新赋值上：它不应该被使用。他们倾向使用 const 来使用所有闭包变量只读，并且他们使用 Ojbect.freeze(..) 或者完全不可变数据结构来防止属性被重新赋值。此外，他们尽量在每个可能的地方减少显式地声明的／追踪的变量，更倾向于值传递———函数链，作为参数被传递的 `return` 值，等等——替代中间值存储。

这本书关于 JavaScript 中的轻量级函数式编程，这是一个我与核心函数式编程群体有分歧的情况。

我认为变量重新赋值当被合理的使用时是相当有用的，它的明确性具有相当有可读性。可以确信能被感受到的是当你能插入一个debugger或者断点，或者能追踪一个观察表达式时调试简单了很多，

### 克隆拷贝

正如我们在第六章学习的，我们预防来自损坏代码可预测性的副作用的最佳方法是我们对待所有状态值为不可变的，无论他们是否真的可变（冻结）与否。

如果你没有使用特别定制的库来提供复杂的不可变数据结构，最简单满足要求的方法：在每次变化前复制你的对象或者数组。

数组浅拷贝很容易：只要使用slice()方法：

```js
var a = [ 1, 2, 3 ];

var b = a.slice();
b.push( 4 );

a;			// [1,2,3]
b;			// [1,2,3,4]
```

对象也可以被相对简单的浅拷贝：

```js
var o = {
	x: 1,
	y: 2
};

// 在 ES2017以后，使用对象的解构：
var p = { ...o };
p.y = 3;

// 在 ES2015以后：
var p = Object.assign( {}, o );
p.y = 3;
```

如果对象或数组中的值是它们这类非基本类型（对象或数组），使用深拷贝你不得不手动遍历每一层来拷贝每个内嵌对象。否则，你将有这些内部对象的共享引用拷贝，这就像给你的程序逻辑造成了一次大破坏。

你是否意识到克隆是可行的只是因为所有的这些状态值是可见的并且可以如此简单地被拷贝？一堆被包装在闭包里的状态会怎么样，你如何克隆这些状态？

那是相当乏味的。基本上，你不得不做一些类似之前我们自定义 `forEach` API的方法：提供一个闭包内层的拥有提取或拷贝隐藏值权限的函数，并在这过程中创建新的等价闭包。

尽管这在理论上是可行的——给读者们的另一个练习——这个实现的操作量远远不及你可能进行的任何真实程序的调整。

当我们遇到表达我们需要可以克隆的状态时，对象具有一个更明显的优势。

### 性能

从实现的角度看，对象有一个比闭包有利的原因，是 JavaScript 对象通常在内存和甚至计算角度是更加轻量的。

但是对这个普遍的断言需要小心：有许多你可以对对象做的事会抹除你从无视闭包转向对象状态追踪获得的好处。

让我们考虑一个情景的两种实现。首先，闭包方式实现：

```js
function StudentRecord(name,major,gpa) {
	return function printStudent(){
		return `${name}, Major: ${major}, GPA: ${gpa.toFixed(1)}`;
	};
}

var student = StudentRecord( "Kyle Simpson", "kyle@some.tld", "CS", 4 );

// later

student();
// Kyle Simpson, Major: CS, GPA: 4.0
```

内部函数 `printStudeng()` 封装了三个变量：`name`，`major`，和` gpa`。它维护这个状态无论我们是否传递引用给这个函数——在这个例子我们称它为 `student()`。

现在看对象（和 `this`）方式：

```js
function StudentRecord(){
	return `${this.name}, Major: ${this.major}, GPA: ${this.gpa.toFixed(1)}`;
}

var student = StudentRecord.bind( {
	name: "Kyle Simpson",
	major: "CS",
	gpa: 4
} );

// later

student();
// Kyle Simpson, Major: CS, GPA: 4.0
```

`student()`  函数，学术上叫做“边界函数”——有一个硬边界 `this` 引用我们传入的对象字面量，因此之后任何调用 `student()`  将使用这个对象作为 `this`，因此可以访问它的封装状态。

两种实现有相同的输出：一个保存状态的函数，但是关于性能，这里的差别是什么。

注意：精准可控地判断 JS 代码片段性能是非常困难的事情。我们在这里不会深入所有的细节，但是我强烈推荐你阅读 “你不知道的 JS：异步和性能”这本书，特别是第六章“性能测试和调优”，来了解细节。

如果你写过一个库来创造持有配对状态的函数——要么用第一个片段中调用 `studentRecord(..)` ，要么用第二个片段中调用 `StudentRecord.bind(..)`的方式——你可能更多的关心它们两的性能怎样。从代码角度，我们可以看到前一种每次创建一个新的函数表达式。第二种使用 `bind(..)`，没有明显的含义。

思考 `bind(..)` 在内部做了什么的一种方式是创建一个闭包来替代函数，像这样：

```js
function bind(orinFn,thisObj) {
	return function boundFn(...args) {
		return origFn.apply( thisObj, args );
	};
}

var student = bind( StudentRecord, { name: "Kyle.." } );
```

在这种方式，看起来我们的场景的两种实现都是创造一个闭包，所以性能看似也是一致的。

但是，内建的 `bind(..)` 工具并没有真正的创造一个闭包来实现这个任务。它只是简单地创建了一个函数，然后手动设置它的内部 `this` 给一个特别的对象。这可能比起我们使用闭包本身是一个更高效的操作。

我们这里讨论的在每次操作上的这种性能优化是不值一提的。但是如果你的库的关键部分被使用了成千上万次甚至更多，它的节省能累加的很快。许多库——Bluebird 就是这样一个例子——已经完成移除闭包使用对象的优化，就是这个意思。

在库的使用案例之外，持有配对状态的函数通常在应用的关键路径发生的次数相对非常少。相比之下，典型的使用是函数加状态——在任意一个片段调用 `student()`——是更加常见的。

如果你的代码中也有给定这样情况的场景，你应该更多地考虑前的性能对比。

历史上的边界函数通常具有一个相当糟糕的性能，但是最近已经被 JS 引擎高度优化。如果你在几年前性能测试过这些不同的方式，很可能跟你现在用最近的引擎重复测试的结果完全不一致。

边界函数现在看起来至少跟同样的封装函数表现的一样好。所以这是另一个支持对象比闭包好的点。

我只想重申：性能观察结果不是绝对的，在一个给定场景下决定什么是最好的是非常复杂的。不要随意使用你从别人那里听到的或者是你从之前一些项目中看到的。小心的决定对象还是闭包更适合这个任务。

## 总结

本章的真理无法被直述。必须阅读本章来寻找它的真理。
