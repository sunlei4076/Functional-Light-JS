# Functional-Light JavaScript
# Chapter 7: Closure vs Object

# 章节7: 闭包 vs 对象

A number of years ago, Anton van Straaten crafted what has become a rather famous and oft-cited [koan](https://www.merriam-webster.com/dictionary/koan) to illustrate and provoke an important tension between closure and objects:

数年前，Anton Van Straaten 创造了一个非常有名且被常常引用的禅理来举例和证实一个闭包和对象之间重要的关系。

> The venerable master Qc Na was walking with his student, Anton. Hoping to
> prompt the master into a discussion, Anton said "Master, I have heard that
> objects are a very good thing - is this true?" Qc Na looked pityingly at
> his student and replied, "Foolish pupil - objects are merely a poor man's
> closures."
> 令人尊敬的大师 Qc Na 曾经和他的学生 Anton 一起散步。Anton 希望激起大师进入一个讨论中，说到： 大师，我曾听说对象是一个非常好的东西，是这样么？ Qc Na 同情地看着他的学生回复到, "愚笨的弟子，对象只不过是可怜人的闭包"
>
> Chastised, Anton took his leave from his master and returned to his cell,
> intent on studying closures. He carefully read the entire "Lambda: The
> Ultimate..." series of papers and its cousins, and implemented a small
> Scheme interpreter with a closure-based object system. He learned much, and
> looked forward to informing his master of his progress.
>
> 被批评后，Anton离开他的师傅会到了自己的住处，致力于学习闭包。他认真的阅读整个 ”匿名函数：终极…"系列论文和它的姐妹篇，并且实践了一个基于闭包系统的小的 Scheme 解析器。他学了很多，盼望展现给他导师他的进步。
>
> On his next walk with Qc Na, Anton attempted to impress his master by
> saying "Master, I have diligently studied the matter, and now understand
> that objects are truly a poor man's closures." Qc Na responded by hitting
> Anton with his stick, saying "When will you learn? Closures are a poor man's
> object." At that moment, Anton became enlightened. 
>
> 当他下一次与 Qc Na 一同散步时，Anton 试着提醒他的导师，说到 “导师，我已经勤奋地学习了这件事，我现在明白了对象真的是可怜人的闭包” ，Qc Na 用棍子戳了戳 Anton 回应到，“你什么时候学的，闭包才是可怜人的对象”。在那一刻， Anton明白了什么。
>
> Anton van Straaten 6/4/2003
>
> http://people.csail.mit.edu/gregs/ll1-discuss-archive-html/msg03277.html

The original posting, while brief, has more context to the origin and motivations, and I strongly suggest you read that post to properly set your mindset for approaching this chapter.

原帖尽管简单，却有更多的内容关于起源和动机，我强烈推荐你阅读帖子来调整你的心态来理解本章。

Many people I've observed read this koan smirk at its clever wit but then move on without it changing much about their thinking. However, the purpose of a koan (from the Bhuddist Zen perspective) is to prod the reader into wrestling with the contradictory truths therein. So, go back and read it again. Now read it again.

我观察到很多人读完这个会对其中的聪明智慧傻笑，接着继续不改变他们的想法。但是，这个禅理（来自 Bhuddist Zen 观点） 是促使读者进入其中对立真相的辩驳中。所以，回去再读一遍，现在再读一遍。

Which is it? Is a closure a poor man's object, or is an object a poor man's closure? Or neither? Or both? Is merely the take-away that closures and objects are in some way equivalent?

到底是哪个？是闭包是可怜人的对象，还是对象是可怜人的闭包？或都不是？或都是？这仅仅带出闭包和对象在某些方面是相同的？

And what does any of this have to do with functional programming? Pull up a chair and ponder for awhile. This chapter will be an interesting detour, an excursion if you will.

还有它们中哪个与函数式编程相关？拉一把椅子过来并且仔细考虑一会儿。如果你愿意，这一章将是一个精彩的迂回之路，一个远足。

## The Same Page

## 达成共识

First, let's make sure we're all on the same page when we refer to closures and objects. We're obviously in the context of how JavaScript deals with these two mechanisms, and specifically talking about simple function closure (see "Keeping Scope" in Chapter 2) and simple objects (collections of key-value pairs).

先确定一点，当我们谈及闭包和对象我们都达成了共识。我们显然在 JavaScript 如何处理这两种机制的环境中，且特制简单函数闭包（见第二章的”保持作用域“）和简单对象（键值对的集合）。

A simple function closure:

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

A simple object:

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

Many people conjur lots of extra things when you mention "closure", such as the asynchronous callbacks or even the module pattern with encapsulation and information hiding. Similarly, "object" brings to mind classes, `this`, prototypes, and a whole slew of other utilities and patterns.

但提到“闭包“时，很多人会想很多额外的事情，例如异步回调甚至是封装和信息隐藏的模块模式。同样，“object”会让人想起类，this，原型和大量其它的工具和模式。

As we go along, we'll carefully address the parts of this external context that matter, but for now, try to just stick to the simplest interpretations of "closure" and "object" -- it'll make this exploration less confusing.

随着深入，我们会需要小心地处理部分额外的相关内容，但是现在，尽量只记住闭包和对象最简单的释义，这会减少很多探索过程中迷惑。



## Look Alike

It may not be obvious how closures and objects are related. So let's explore their similarities first.

To frame this discussion, let me just briefly assert two things:

闭包和对象之间的关系可能不是那么明显。让我们先来探究它们之间的相似点。为了给这次讨论一个基调，让我简述两件事：

1. A programming language without closures can simulate them with objects instead.

   一个没有闭包的编程语言可以用对象来模拟闭包。

2. A programming language without objects can simulate them with closures instead.

   一个没有对象的编程语言可以用闭包来模拟对象。

In other words, we can think of closures and objects as two different representations of a thing.

换句话说，我们可以认为闭包和对象是一样东西的两种表达方式。

### State

Consider this code from above:

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

Both the scope closed over by `inner()` and the object `obj` contain two elements of state: `one` with value `1` and `two` with value `2`. Syntactically and mechanically, these representations of state are different. But conceptually, they're actually quite similar.

inner()和obj对象持有的作用域都包含了两个元素状态：值为1的one和值为2的two。从语法和机制来说，这两种声明状态是不同的。但概念上，他们的确相当相似。

As a matter of fact, it's fairly straightforward to represent an object as a closure, or a closure as an object. Go ahead, try it yourself:

事实上，表达一个对象为闭包形式，或闭包为对象形式是相当简单的。接下来，尝试一下：

```js
var point = {
	x: 10,
	y: 12,
	z: 14
};
```

Did you come up with something like?

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

**Note:** The `inner()` function creates and returns a new array (aka, an object!) each time it's called. That's because JS doesn't afford us any capability to `return` multiple values without encapsulating them in an object. That's not technically a violation of our object-as-closure task, because it's just an implementation detail of exposing/transporting values; the state tracking itself is still object-free. With ES6+ array destructuring, we can declaratively ignore this temporary intermediate array on the other side: `var [x,y,z] = point()`. From a developer ergonomics perspective, the values are stored individually and tracked via closure instead of objects.

说明：每次被调用时 inner() 方法创造并返回了一个新的数组（亦然是一个对象）。这是因为JS不提供我们返回多个数据却不包装在一个对象中的能力。这并不是严格意义上一个违反我们对象类似闭包的工作，因为这只是一个暴露／运输具体值的实现，我们可以声明地忽视这个临时中间对象通过另一种方式：var [x,y,z] = point()。从开发者工程学角度，值应该被单独存储并且通过闭包而不是对象追踪。

What if we have nested objects?

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

We could represent that same kind of state with nested closures:

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

Let's practice going the other direction, from closure to object:

让我们尝试另一个从闭包到对象的方向：

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

`distFromPoint(..)` is closed over `x1` and `y1`, but we could instead explicitly pass those values as an object:

distFromPoint(..) 封装了 x1 和 y1 ，但是我们也可以通过传入一个具体的对象作为值：

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

The `point` object state explicitly passed in replaces the closure that implicitly held that state.

point 对象状态明确地被传入替换了闭包隐式持有的状态。

#### Behavior, Too!

行为，也是一样

It's not just that objects and closures represent ways to express collections of state, but also that they can include behavior via functions/methods. Bundling data with its behavior has a fancy name: encapsulation.

对象和闭包不仅仅是表达状态集合的方式，但是他们也可以包含函数或者方法。将数据和行为捆绑有一个充满想象力的名字：封装。

Consider:

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

The inner function `happyBirthday()` has closure over `name` and `age` so that the functionality therein is kept with the state.

内部函数 happyBirthday() 封闭了 name 和 age 所以内部的函数也持有了这个状态。 

We can achieve that same capability with a `this` binding to an object:

我们也可以通过对象内部的 this 绑定来获取这个能力：

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

We're still expressing the encapsulation of state data with the `happyBirthday()` function, but with an object instead of a closure. And we don't have to explicitly pass in an object to a function (as with earlier examples); JavaScript's `this` binding easily creates an implicit binding.

我们仍然通过 happyBrithday() 函数来表达对状态数据的封装，但是用对象代替了闭包。同时我们没有显式给函数传递一个对象（如同先前的例子）；JavaScript的this绑定可以创造一个隐式的绑定。

Another way to analyze this relationship: a closure associates a single function with a set of state, whereas an object holding the same state can have any number of functions to operate on that state.

另一个分析它们关系的方面：闭包结合了一个包含了一系列声明的函数，而对象持有同样的声明却可以用任意数量的函数去操作这些状态。

As a matter of fact, you could even expose multiple methods with a single closure as the interface. Consider a traditional object with two methods:

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

Just using closure without objects, we could represent this program as:

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

While these programs look and feel a bit different ergonomically, they're actually just different implementation variations of the same program behavior.

尽管这些程序看起来感觉有点反人类， 但它们实际上只是相同程序的不同实现。

### (Im)mutability

### (不)可变

Many people will initially think that closures and objects behave differently with respect to mutability; closures protect from external mutation while objects do not. But, it turns out, both forms have identical mutation behavior.

许多人最初都认为闭包和对象行为的差别源于可变性；闭包会阻止来自外部的变化而对象则不然。但是，结果是，这两种形式都有典型的可变行为。

That's because what we care about, as discussed in Chapter 6, is **value** mutability, and this is a characteristic of the value itself, regardless of where or how it's assigned.

正如第六章讨论的，这是因为我们关心的是值的可变性，值可变是值本身的特性，不在于在哪里或者如何被赋值的。

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

The value stored in the `x` lexical variable inside `outer()` is immutable -- remember, primitives like `2` are by definition immutable. But the value referenced by `y`, an array, is definitely mutable. The exact same goes for the `x` and `y` properties on `xyPublic`.

在 outer() 中字面变量x存储的值是不可变的 — 记住，定义的原始值如2是不可变得。但是y的引用值，一个数组，绝对是可变的。这点对于xyPublic中的 x 和 y 属性也是完全相同的。

We can reinforce the point that objects and closures have no bearing on mutability by pointing out that `y` is itself an array, and thus we need to break this example down further:

通过指出 y 本身是个数组我们可以强调对象和闭包在可变这点上没有关系，因此我们需要将这个例子继续拆解：

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

If you think about it as "turtles (aka, objects) all the way down", at the lowest level, all state data is primitives, and all primitives are value-immutable.

如果你认为这个如同 "海龟 (又称, 对象) 背地球"，在最底层，所有的状态数据都是基本类型，而所有基本类型都是不可变值。

Whether you represent this state with nested objects, or with nested closures, the values being held are all immutable.

不论是用嵌套对象还是嵌套闭包代表状态，这些被持有的值都是不可变的。

### Isomorphic

###同构

The term "isomorphic" gets thrown around a lot in JavaScript these days, and it's usually used to refer to code that can be used/shared in both the server and the browser. I wrote a blog post awhile back that calls bogus on that usage of this word "isomorphic", which actually has an explicit and important meaning that's being clouded.

同构这个概念最近在 JavaScript 圈被经常提出，它通常被用来指代码可以同时被服务端和浏览器端使用／分享。 我不久以前写了一篇博文说明这种对同构这个词的使用是错误的，隐藏了它实际上确切和重要的意思。

Here's some selections from a part of that post:

这里我是博文部分的节选：

> What does isomorphic mean? Well, we could talk about it in mathematic terms, or sociology, or biology. The general notion of isomorphism is that you have two things which are similar in structure but not the same.
>
> 同构的意思是什么？当然，我们可以用数学词汇，社会学或者生物学讨论它。同构最普遍的概念是你有两个类似但是不相同的结构。
>
> In all those usages, isomorphism is differentiated from equality in this way: two values are equal if they’re exactly identical in all ways, but they are isomorphic if they are represented differently but still have a 1-to-1, bi-directional mapping relationship.
>
> 在这些所有的惯用法中，同构和相等的区别在这里：如果两个值在各方面完全一致那么它们相等，但是如果它们表现不一致却仍有一对一或者双向映射的关系那么它们是同构。
>
> In other words, two things A and B would be isomorphic if you could map (convert) from A to B and then go back to A with the inverse mapping.
>
> 换而言之，两件事物A和B如果你能够映射（转化）A到B并且能够通过反向映射回到A那么它们就是同构。

Recall in "Brief Math Review" in Chapter 2, we discussed the mathematical definition of a function as being a mapping between inputs and outputs. We pointed out this is technically called a morphism. An isomorphism is a special case of bijective (aka, 2-way) morphism that requires not only that the mapping must be able to go in either direction, but also that the behavior is identical in either form.

回想第二章的简单数学回顾，我们讨论了函数的数学定义是一个输入和输出之间的映射。我们指出这在学术上称为摹式。同构是双射（双向）摹式的特殊案例，它需要不仅仅映射必须可以从任意一边完成，而且在任何形式行为完全相同。

But instead of thinking about numbers, let's relate isomorphism to code. Again quoting my blog post:

 不去思考这些关于数字的，让我们关联同构到代码。再一次引用我的博文：

> [W]hat would isomorphic JS be if there were such a thing? Well, it could be that you have one set of JS code that is converted to another set of JS code, and that (importantly) you could convert from the latter back to the former if you wanted.
>
> 如果有的话同构 JS 是怎么样的？它可能是你有一集合的 JS 代码转化为了另一集合的 JS 代码，并且（重要地）如果你想你可以把转化后的转为之前的。

As we asserted earlier with our examples of closures-as-objects and objects-as-closures, these representative alternations go either way. In this respect, they are isomorphisms of each other.

正如我们之前通过闭包如同对象和对象如同闭包为例声称的一样，这种可以通过两者中的任何一个方式交替展现的。就一点来说，它们互为同构。

Put simply, closures and objects are isomorphic representations of state (and its associated functionality).

简而言之，闭包和对象是状态的同构表示 （和它们相关的功能性）。

The next time you hear someone say "X is isomorphic with Y", what they mean is, "X and Y can be converted from either one to the other in either direction, and maintain the same behavior regardless."

下次你听到谁说 ”X 与 Y 是同构的“，他们的意思是， ” X 和 Y 可以从两种中的任意一方转化到另一方， 并且无论怎么都保持了相同的特性。“

### Under The Hood

内部结构

So, we can think of objects as an isomorphic representation of closures from the perspective of code we could write. But we can also observe that a closure system could actually be implemented -- and likely is -- with objects!

所以，我们可以从我们写的代码角度想象对象是闭包的一种同构展示。但我们也可以观察闭包系统可以被实现—最合适的就是—通过对象。

Think about it this way: in the following code, how is JS keeping track of the `x` variable for `inner()` to keep referencing, well after `outer()` has already run?

在这方面考虑一下：在如下的代码中， 在 outer() 已经运行后，JS 如何为了 inner（）保持引用来持续记录x变量？

```js
function outer() {
	var x = 1;

	return function inner(){
		return x;
	};
}
```

We could imagine that the scope -- the set of all variables defined -- of `outer()` is implemented as an object with properties. So, conceptually, somewhere in memory, there's something like:

我们会想到作用域，outer（）的所有变量定义通过一个变量的属性定义。因此，从概念上讲，在内存中的某个地方，是类似这样的。

```js
scopeOfOuter = {
	x: 1
};
```

And then for the `inner()` function, when created, it gets an (empty) scope object called `scopeOfInner`, which is linked via its `[[Prototype]]` to the `scopeOfOuter` object, sorta like this:

接下来对于 inner() 函数，但创建时，它获得了一个叫做scopeOfInner的（空）作用域对象，这个对象被其[[Prototype]] 连接到 scopeOfOuter 对象，近似这个：

```js
scopeOfInner = {};
Object.setPrototypeOf( scopeOfInner, scopeOfOuter );
```

Then, inside `inner()`, when it makes reference to the lexical variable `x`, it's actually more like:

接着，内部的 inner(), 当他建立词法变量X的引用时，实际跟像这样：

```js
return scopeOfInner.x;
```

`scopeOfInner` doesn't have an `x` property, but it's `[[Prototype]]`-linked to `scopeOfOuter`, which does have an `x` property. Accessing `scopeOfOuter.x` via prototype delegation results in the `1` value being returned.

scopeOfInner 并没有一个 x 的属性，当他的[[Prototype]]连接到拥有 x 属性的 scopeOfOuter。访问 scopeOfOuter.x  通过原型委托返回 1 作为结果。

In this way, we can sorta see why the scope of `outer()` is preserved (via closure) even after it finishes: because the `scopeOfInner` object is linked to the `scopeOfOuter` object, thereby keeping that object and its properties alive and well.

这样，我们可以近似认为为什么 outer() 的作用域甚至在当它执行完都被保留（通过闭包）：因为 scopeOfInner 对象连接到 scopeOfOuter 对象，因此，保持了这个对象和它的属性很好的存在。

Now, this is all conceptual. I'm not literally saying the JS engine uses objects and prototypes. But it's entirely plausible that it *could* work similarly.

现在，这都只是概念。我没有字面的说 JS 引擎使用对象和原型。但是它可以类似这样工作是完全合理的。

Many languages do in fact implement closures via objects. And other languages implement objects in terms of closures. But we'll let the reader use their imagination on how that would work.

许多语言实际上通过对象实现了闭包。另一些语言用闭包的概念实现了对象。但是我们让读者使用他们的想象力思考这是如何工作的。

## Two Roads Diverged In A Wood...

一块木头的两路分支

So closures and objects are equivalent, right? Not quite. I bet they're more similar than you thought before you started this chapter, but they still have important differences.

所以闭包和对象是等价的，对吗？不完全是，我打赌它们比你在读本章前想的更加想家相似，但是它们仍有重要的区别点。

These differences should not be viewed as weaknesses or arguments against usage; that's the wrong perspective. They should be viewed as features and advantages that make one or the other more suitable (and readable!) for a given task.

这些区别点不应当被视作缺点或者不利于使用的论点；这是错误的观点。它们应当被视作特性和优点使一个（闭包或对象）或另一个更适合做一个给定的任务。

### Structural Mutability

结构可变性

Conceptually, the structure of a closure is not mutable.

从概念上讲，闭包的结构不是可变的。

In other words, you can never add to or remove state from a closure. Closure is a characteristic of where variables are declared (fixed at author/compile time), and is not sensitive to any runtime conditions -- assuming you use strict mode and/or avoid using cheats like `eval(..)`, of course!

换而言之，你永远不能从闭包添加或移除状态。闭包是一个表示对象在哪里声明的特性（被固定在编写／编译时间），并且不受任何条件的影响 — 假设你使用严格模式并且／或者避免使用作弊手段例如eval(..)，当然。

**Note:** The JS engine could technically cull a closure to weed out any variables in its scope that are no longer going to be used, but this is an advanced optimization that's transparent to the developer. Whether the engine actually does these kinds of optimizations, I think it's safest for the developer to assume that closure is per-scope rather than per-variable. If you don't want it to stay around, don't close over it!

注意：JS引擎可以技术上过滤一个对象来清除其作用域中不再被使用的变量，但是这是一个对于开发者透明的先进优化。无论引擎是否实际做了这类优化，我认为对于开发者假设闭包是作用域优先而不是变量优先是最安全的。如果你不想保留它，就不要封闭它（在闭包里）！

However, objects by default are quite mutable. You can freely add or remove (`delete`) properties/indices from an object, as long as that object hasn't been frozen (`Object.freeze(..)`).

但是，对象默认是完全可变的，你可以自由的添加或者移除一个对象的属性／索引，只要对象没有被冻结（Object.freeze(..)）

It may be an advantage of the code to be able to track more (or less!) state depending on the runtime conditions in the program.

这或许是代码可以根据程序中运行时条件追踪跟多（或更少）状态的优势。

For example, let's imagine tracking the keypress events in a game. Almost certainly, you'll think about using an array to do this:

举个例子，让我们思考追踪游戏中的按键事件。几乎可以肯定，你会考虑使用一个数组来做这件事：

```js
function trackEvent(evt,keypresses = []) {
	return keypresses.concat( evt );
}

var keypresses = trackEvent( newEvent1 );

keypresses = trackEvent( newEvent2, keypresses );
```

**Note:** Did you spot why I used `concat(..)` instead of `push(..)`ing directly to `keypresses`? Because in FP, we typically want to treat arrays as immutable data structures that can be recreated and added to, but not directly changed. We trade out the evil of side-effects for an explicit reassignment (more on that later).

注意：你能否认出为什么我使用 concat(..) 而不是直接 push(..) 像 keypresses 数组？因为在函数式编程中，我们通常希望对待数组如同不可变数据结构，可以被创建和添加，但不能直接改变。我们

Though we're not changing the structure of the array, we could if we wanted to. More on this in a moment.

But an array is not the only way to track this growing "list" of `evt` objects. We could use closure:

尽管我们不在改变数组的结构，但当我们希望时我们也可以。稍后详细介绍。数组不是仅有的方式来记录这个事件对象的增长“列表”。我们可以使用闭包：

```js
function trackEvent(evt,keypresses = () => []) {
	return function newKeypresses() {
		return [ ...keypresses(), evt ];
	};
}

var keypresses = trackEvent( newEvent1 );

keypresses = trackEvent( newEvent2, keypresses );
```

Do you spot what's happening here?

你看出这里发生了什么？

Each time we add a new event to the "list", we create a new closure wrapped around the existing `keypresses()` function (closure), which captures the current `evt`. When we call the `keypresses()` function, it will successively call all the nested functions, building up an intermediate array of all the individually closed-over `evt` objects. Again, closure is the mechanism that's tracking all the state; the array you see is only an implementation detail of needing a way to return multiple values from a function.

每次我们添加一个新的事件到这个“列表”，我们创建了一个包装了现有 keypresses() 方法（闭包）的新闭包，这个新闭包捕获了当前的 evt 。当我们调用 keypresses() 函数，它将成功的调用所有的内部方法，创建一个包含所有独立封装的evt对象的中间数组。再次说明，数组是一个追踪所有状态的机制；这个你看到的数组只是一个对于需要一个方法来返回函数中多个值的具体实现。

So which one is better suited for our task? No surprise here, the array approach is probably a lot more appropriate. The structural immutability of a closure means our only option is to wrap more closure around it. Objects are by default extensible, so we can just grow the array as needed.

所以那一个更适合我们的任务？毫无意外，数组方法可能更合适一些。闭包的不可变结构意味着我们的唯一选项饰封装跟多的闭包在里面。对象默认是可扩展的，所以我们需要增长对象就足够了。

By the way, even though I'm presenting this structural (im)mutability as a clear difference between closure and object, the way we're using the object as an immutable value is actually more similar than dislike.

顺便一提，尽管我们表现出结构不可变或可变是一个闭包和对象之间的明显区别，我们使用对象作为一个不可变数据的方法实际上更亲切而非厌恶。

Creating a new array (via `concat(..)`) for each addition to the array is treating the array as structurally immutable, which is conceptually symmetrical to closure being structurally immutable by its very design.

数组每次添加就创造一个新数组（通过concat(..)）就是把数组对待为结构不可变，这个概念上对等于通过适当的设计使闭包结构上不可变。

### Privacy

私有

Probably one of the first differences you think of when analyzing closure vs object is that closure offers "privacy" of state through nested lexical scoping, whereas objects expose everything as public properties. Such privacy has a fancy name: information hiding.

当对比分析闭包和对象时可能你思考的第一个区分点就是闭包通过词法作用域提供“私有”状态，而对象将一切做为公共属性暴露。这种私有有一个精致的名字：信息隐藏。

Consider lexical closure hiding:

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

Now the same state in public:

现在同样的状态公开：

```js
var xPublic = {
	x: 1
};

xPublic.x;			// 1
```

There's some obvious differences around general software engineering principles -- consider abstraction, the module pattern with public and private APIs, etc -- but let's try to constrain our discussion to the perspective of FP; this is, after all, a book about functional programming!

这里有一些明显的区别在普遍的软件工程原理 — 考虑下抽象，这种模块模式有着公开和私有APIs等等。但是让我们束缚我们的讨论在FP角度，毕竟，这是一本关于函数式编程的书！

#### Visibility

可见性

It may seem that the ability to hide information is a desired characteristic of state tracking, but I believe the FPer might argue the opposite.

可能看起来对于状态追踪隐藏信息的能力是一个被渴望的特性，但是我认为函数式编程者可能持反对观点。

One of the advantages of managing state as public properties on an object is that it's easier to enumerate (and iterate!) all the data in your state. Imagine you wanted to process each keypress event (from the earlier example) to save it to a database, using a utility like:

在一个对象中管理状态为公开属性的一个优点是这使你状态中的所有数据更容易枚举（迭代）。思考下你想访问每一个按键事件（从之前的那个例子）并且存储到一个数据库，使用一个这样的工具：

```js
function recordKeypress(keypressEvt) {
	// database utility
	DB.store( "keypress-events", keypressEvt );
}
```

If you already have an array -- just an object with public numerically-named properties -- this is very straightforward using a built-in JS array utility `forEach(..)`:

如果你已经有一个数组，正好是一个拥有公开的用数字命名属性的对象— 非常直接的使用 JS 对象的内建工具 forEach(..)：

```js
keypresses.forEach( recordKeypress );
```

But, if the list of keypresses is hidden inside closure, you'll have to expose a utility on the public API of the closure with privileged access to the hidden data.

但是，如果按键列表被隐藏在一个闭包里，你不得不在闭包内暴露一个享有特权访问数据的公开API工具。

For example, we can give our closure-`keypresses` example its own `forEach`,  like built-in arrays have:

举例而说，我可以给我们的闭包- keypresses例子自己的forEach，像数组内建有的：

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

The visibility of an object's state data makes using it more straightforward, whereas closure obscures the state making us work harder to process it.

对象状态数据的可见性让我们能更直接的使用它，而闭包遮掩状态让我们更艰难的处理它。

#### Change Control

变更控制

If the lexical variable `x` is hidden inside a closure, the only code that has the freedom to reassign it is also inside that closure; it's impossible to modify `x` from the outside.

如果词法变量被隐藏在一个闭包中，只有闭包内部的代码才能自由的重新赋值，在外部修改 x 是不可能的。

As we saw in Chapter 6, that fact alone improves the readability of code by reducing the surface area that the reader must consider to predict the behavior of any given variable.

正如我们在第6章看到的，提升代码可读性的唯一真相就是减少表面掩盖，读者必须可以预见到每一个给定变量的行为。

The local proximity of lexical reassignment is a big reason why I don't find `const` as a feature that helpful. Scopes (and thus closures) should in general be pretty small, and that means there will only be a few lines of code that can affect reassignment. In `outer()` above, we can quickly inspect to see that no line of code reassigns `x`, so for all intents and purposes it's acting as a constant.

词法（作用域）重新赋值局部邻接的是一个让我找不到const特性有用的大问题。作用域（）通常应该尽可能小，这意味着重新赋值只会影响几行代码。在上面的 outer() 中，我们可以快速地检查没有一行代码重设了x，所有意图和目的表现像一个常量。

This kind of guarantee is a powerful contributor to our confidence in the purity of a function, for example.

这类保证对于例如我们对函数纯净的信任是一个重要的保证。

On the other hand, `xPublic.x` is a public property, and any part of the program that gets a reference to `xPublic` has the ability, by default, to reassign `xPublic.x` to some other value. That's a lot more lines of code to consider!

反而言之，xPublic.x 是一个公共属性，程序的任何部分都拥有 xPublic 的一份引用，有了默认重设 xPublic.x 到别的值的能力。这会让很多行代码需要被考虑。

That's why in Chapter 6, we looked at `Object.freeze(..)` as a quick-n-dirty means of making all of an object's properties read-only (`writable: false`), so that they can't be reassigned unpredictably.

这是为什么在第六章，我们视 Object.freeze(..) 为一个龌龊的方案使所有的对象属性只读（可写性：非），所以它们不能被不可预测的重设。

Unfortunately, `Object.freeze(..)` is both all-or-nothing and irreversible.

不幸的是，Object.freeze(..) 是极端且不可逆的。

With closure, you have some code with the privilege to change, and the rest of the program is restricted. When you freeze an object, no part of the code will be able to reassign. Moreover, once an object is frozen, it can't be thawed out, so the properties will remain read-only for the duration of the program.

使用闭包，我们可以有一些代码有权限改变，剩余的程序是受限的。当我们冻结一个对象，代码中没有任何部分可以被重设。此外，一定一个对象被冻结，它不能被解冻，所以所有属性在程序运行期间都保持只读。

In places where I want to allow reassignment but restrict its surface area, closures are a more convenient and flexible form than objects. In places where I want no reassignment, a frozen object is a lot more convenient than repeating `const` declarations all over my function.

在我想允许重新赋值但是在表面限制的地方，闭包比起对象更方便和灵活。在我不想重新赋值的地方，一个冻结的对象比起重复 const 声明在我所有的函数更方便一些。

Many FPers take a hard-line stance on reassignment: it shouldn't be used. They will tend to use `const` to make all closure variables read-only, and they'll use `Object.freeze(..)` or full immutable data structures to prevent property reassignment. Moreover, they'll try to reduce the amount of explicitly declared/tracked variables and properties wherever possible, perferring value transfer -- function chains, `return` value passed as argument, etc -- instead of intermediate value storage.

许多函数式编程者采取了一个硬派立场在重新赋值上：这不应该被使用。他们倾向使用 const 来使用所有闭包变量只读，并且他们使用 Ojbect.freeze(..) 或者完全不可变数据结构来放置属性被重新赋值。此外，他们尽量在每个可能的地方减少明确的／可记录的声明变量，更倾向于值传递 — 函数链，返回作为参数传来的值，等等 — 替代中间值存储。

This book is about "functional light" programming in JavaScript, and this is one of those cases where I diverge from the core FP crowd.

这本书关于 JavaScript 中的轻量级函数式编程，这是一个我与核心函数式编程群众有分歧的情况。

I think variable reassignment can be quite useful and, when used approriately, quite readable in its explicitness. It's certainly been by experience that debugging is a lot easier when you can insert a `debugger` or breakpoint, or track a watch expression.

我认为变量重新赋值是相当有用的，当被合理的使用，它的明确性是相当有可读性的。可以确信在调试上的体验更简单，当你在插入一个debugger或者断点，或者追踪一个观察表达式。

### Cloning State

克隆拷贝

As we learned in Chapter 6, one of the best ways we prevent side effects from eroding the predictability of our code is to make sure we treat all state values as immutable, regardless of whether they are actually immutable (frozen) or not.

正如我们在第六章学习的，我们预防来自损坏代码可预测性的副作用的最佳方法是我们对待所有状态值为不可变的，无论他们是否真的可变（冻结）与否。

If you're not using a purpose-built library to provide sophisticated immutable data structures, the simplest approach will suffice: duplicate your objects/arrays each time before making a change.

如果你没有使用特别定制的库来提供复杂的不可变数据结构，最简单满足要求的方法：在每次变化前复制你的对象或者数组。

Arrays are easy to clone shallowly: just use the `slice()` method:

数组浅拷贝很容易：只要使用slice()方法：

```js
var a = [ 1, 2, 3 ];

var b = a.slice();
b.push( 4 );

a;			// [1,2,3]
b;			// [1,2,3,4]
```

Objects can be shallow-cloned relatively easily too:

对象也可以被相对简单的浅拷贝：

```js
var o = {
	x: 1,
	y: 2
};

// in ES2017+, using object spread:
// 在 ES2017以后，使用对象的解构：
var p = { ...o };
p.y = 3;

// in ES2015+:
// 在 ES2015以后：
var p = Object.assign( {}, o );
p.y = 3;
```

If the values in an object/array are themselves non-primitives (objects/arrays), to get deep cloning you'll have to walk each layer manually to clone each nested object. Otherwise, you'll have copies of shared references to those sub-objects, and that's likely to create havoc in your program logic.

如果对象或数组中的值是它们这类非基本类型（对象或数组），使用深拷贝，你不得不遍历手动遍历每一层来拷贝每个内嵌对象。否则，你将有这些内部对象的共享引用拷贝，这就像给你的程序逻辑造成了一次大破坏。

Did you notice that this cloning is possible only because all these state values are visible and can thus be easily copied? What about a set of state wrapped up in a closure; how would you clone that state?

你是否意识到克隆是可行的只是因为所有的这些状态值是可见的并且可以如此简单地被拷贝？一群被包装在闭包的状态会怎么样，你如何克隆这些状态。

That's much more tedious. Essentially, you'd have to do something similar to our custom `forEach` API method earlier: provide a function inside each layer of the closure with the privilege to extract/copy the hidden values, creating new equivalent closures along the way.

这会更加的令人厌烦。基本上，你不得不做一些类似之前我们自定义 forEach API的方法：提供一个闭包内层的拥有提取或拷贝隐藏值权限的函数，并在这过程中创建新的等价闭包。

Even though that's theoretically possible -- another exercise for the reader! -- it's far less practical to implement than you're likely to justify for any real program.

尽管这在理论上是可行的 — 给读者们的另一个练习 — 

Objects have a clear advantage when it comes to representing state that we need to be able to clone.

当我们谈到表达我们需要可以克隆的状态时对象有一个更明显的优势。

### Performance

性能

One reason objects may be favored over closures, from an implementation perspective, is that in JavaScript objects are often lighter-weight in terms of memory and even computation.

从实现的角度看，对象有一个有利于闭包的原因，是 JavaScript 对象通常在内存和甚至计算角度是更加轻量的。

But be careful with that as a general assertion: there are plenty of things you can do with objects that will erase any performance gains you may get from ignoring closure and moving to object-based state tracking.

但是对这个普遍的断言需要小心：有许多你可以在对象做的事会抹除你从无视闭包转向对象状态追踪获得的好处。

Let's consider a scenario with both implementations. First, the closure-style implementation:

让我们考虑这两种实现的一个情景。首先，闭包方式实现：

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

The inner function `printStudent()` closes over three variables: `name`, `major`, and `gpa`. It maintains this state wherever we transfer a reference to that function -- we call it `student()` in this example.

内部函数 printStudeng() 封装了三个变量：name，major，和 gpa。它维护这个状态无论我们是否传递引用给这个函数 — 在这个例子我们称它为 student()。

Now for the object (and `this`) approach:

现在看对象（和this）方式：

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

The `student()` function -- technically referred to as a "bound function" -- has a hard-bound `this` reference to the object literal we passed in, such that any later call to `student()` will use that object for it `this`, and thus be able to access its encapsulated state.

studeng() 函数，学术上叫做“边界函数” —有一个硬边界this引用我们传入的对象字面量，因此之后任何调用student() 将使用这个对象作为this，因此可以访问它的封装状态。

Both implemenations have the same outcome: a function with preserved state. But what about the performance; what differences will there be?

两种实现有相同的输出：一个保存状态的函数，但是关于性能，这里的差别是什么。

**Note:** Accurately and actionably judging performance of a snippet of JS code is a very dodgy affair. We won't get into all the details here, but I urge you to read the "You Don't Know JS: Async & Performance" book, specifically Chapter 6 "Benchmarking & Tuning", for more details.

注意：精准可用的判断 JS 代码片段性能是非常困难的事情。我们在这里不会深入所有的细节，但是我强烈推荐你阅读 “你不知道的 JavaScript：异步和性能”这本书，特别是第六章 “性能测试和调优”，来了解细节。

If you were writing a library that created a pairing of state with its function -- either the call to `StudentRecord(..)` in the first snippet or the call to `StudentRecord.bind(..)` in the second snippet -- you're likely to care most about how those two perform. Inspecting the code, we can see that the former has to create a new function expression each time. The second one uses `bind(..)`, which is not as obvious in its implications.

如果你写过一个库来创造一个一对状态的函数 — 两种中的一个在第一个片段叫做 studentRecord(..) 另一个在第二个片段叫做 StudentRecord.bind(..) — 你可能更多的关心它们两的性能怎样。从代码角度，我们可以看到前一种每次创建一个新的函数表达式。第二种使用 bind(..)，没有明显的含义。

One way to think about what `bind(..)` does under the covers is that it creates a closure over a function, like this:

思考bind(..)在内部做了什么的一种方式是创建一个闭包来替代函数，像这样：

```js
function bind(orinFn,thisObj) {
	return function boundFn(...args) {
		return origFn.apply( thisObj, args );
	};
}

var student = bind( StudentRecord, { name: "Kyle.." } );
```

In this way, it looks like both implementations of our scenario create a closure, so the performance is likely to be about the same.

在这种方式，看起来我们的场景的两种实现都是创造一个闭包，所以性能看似也是一致的。

However, the built-in `bind(..)` utility doesn't really have to create a closure to accomplish the task. It simply creates a function and manually sets its internal `this` to the specified object. That's potentially a more efficient operation than if we did the closure ourselves.

但是，内建的 bind(..) 工具并没有真正的创造一个闭包来实现这个任务。它知识简单地创建了一个函数，然后手动设置它的内部this给一个特别的对象。这可能比起我们使用闭包本身是一个更高效的操作。

The kind of performance savings we're talking about here is miniscule on an individual operation. But if your library's critical path is doing this hundreds or thousands of times or more, that savings can add up quickly. Many libraries -- Bluebird being one such example -- have ended up optimizing by removing closures and going with objects, in exactly this means.

我们这里讨论的在每个操作上的这类性能优化是不值一提的。但是如果你的库的关键部分被使用了成千上万次甚至更多，它的节省能累加的很快。许多库 — Bluebird就是这样一个例子— 已经完成移除闭包使用对象的优化，就是这个意思。

Outside of the library use-case, the pairing of the state with its function usually only happens relatively few times in the critical path of an application. By contrast, typically the usage of the function+state -- calling `student()` in either snippet -- is more common.

在库的使用案例之外，配对状态在一个函数通常在应用的关键路径发生的次数想对非常少。相比之下，典型的使用是函数加状态 — 在任意一个片段调用 student() — 更加常见。

If that's the case for some given situation in your code, you should probably care more about the performance of the latter versus the former.

如果你的代码中也有给定这样情况的场景，你应该更多的考虑前后对比的性能。

Bound functions have historically had pretty lousy performance in general, but have recently been much more highly optimized by JS engines. If you benchmarked these variations a couple of years ago, it's entirely possible you'd get different results repeating the same test with the latest engines.

边界函数在历史上通常有一个相当糟糕的性能，但是最近已经被 JS 引擎高度优化。如果你几年前性能测试这些不同方式，很可能跟你现在用最近的引擎重复测试的结果完全不一致。

A bound function is now likely to perform at least as good if not better as the equivalent closed-over function. So that's another tick in favor of objects over closures.

边界函数现在看起来就算不更好也跟同样的封装函数表现的一样好。所以另一个小技巧就是使用对象来代替闭包。

I just want to reiterate: these performance observations are not absolutes, and the determination of what's best for a given scenario is very complex. Do not just casually apply what you've heard from others or even what you've seen on some other earlier project. Carefully examine whether objects or closures are appropriately efficient for the task.

我只想重申：性能观察结果不是绝对的，在一个给点场景下决定什么是最好的是非常复杂的。不要随意使用你从别人那里听到的或者是你从之前一些项目中看到的。小心的决定对象还是闭包跟适合这个任务。

## Summary

总结

The truth of this chapter cannot be written out. One must read this chapter to find its truth.

本章的真相无法被写出。必须阅读本章来寻找它的真相。
