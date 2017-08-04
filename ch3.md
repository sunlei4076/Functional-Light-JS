# JavaScript 轻量级函数式编程
# 第 3 章：管理函数的输入（Inputs）

在第 2 章的 “函数输入” 小节中，我们聊到了函数形参（parameters）和实参（arguments）的基本知识，实际上还了解到一些能简化其使用方式的语法技巧，比如 `...` 操作符和解构（destructuring）。

在那个讨论中，我建议尽可能设计单一形参的函数。但实际上你不能每次都做到，而且也不能每次都掌控你的函数签名（译注：JS 中，函数签名一般包含函数名和形参等函数关键信息，例如 `foo(a, b = 1, c)`）。

现在，我们把注意力放在更复杂、强大的模式上，以便讨论处在这些场景下的函数输入。

## 立即传参和稍后传参

如果一个函数接收多个实参，你可能会想先指定部分实参，余下的稍后再指定。

来看这个函数：

```js
function ajax(url,data,callback) {
	// ..
}
```

想象一个场景，你要发起多个已知 URL 的 API 请求，但这些请求的数据和处理响应信息的回调函数要稍后才能知道。

当然，你可以等到这些东西都确定后再发起 `ajax(..)` 请求，并且到那时再引用全局 URL 常量。但我们还有另一种选择，就是创建一个已经预设 `url` 实参的函数引用。

我们将创建一个新函数，其内部仍然发起 `ajax(..)` 请求，此外在等待接收另外两个实参的同时，我们手动将 `ajax(..)` 第一个实参设置成你关心的 API 地址。

```js
function getPerson(data,cb) {
	ajax( "http://some.api/person", data, cb );
}

function getOrder(data,cb) {
	ajax( "http://some.api/order", data, cb );
}
```

手动指定这些外层函数当然是完全有可能的，但这可能会变得冗长乏味，特别是不同的预设实参还会变化的时候，譬如：

```js
function getCurrentUser(cb) {
	getPerson( { user: CURRENT_USER_ID }, cb );
}
```

函数式编程者（简称 “FPer”）习惯于在重复做同一种事情的地方找到模式，并试着将这些行为转换为逻辑可重用的实用函数。实际上，该行为肯定已是大多数读者的本能反应了，所以这并非函数式编程（简称 “FP” ）独有。但是，对函数式编程而言，这个行为的重要性是毋庸置疑的。

为了构思这个用于实参预设的实用函数，我们不仅要着眼于之前提到的手动实现方式，还要在概念上审视一下到底发生了什么。

用一句话来说明发生的事情：`getOrder(data,cb)` 是 `ajax(url,data,cb)` 函数的**偏函数**。该术语代表的概念是：在函数调用现场（function call-site），将实参**应用（apply）**于形参。如你所见，我们一开始仅应用了部分实参 —— 具体是将实参应用到 `url` 形参 —— 剩下的实参稍后再应用。

关于该模式更正式的说法是：偏函数严格来讲是一个减少函数参数个数（arity）的过程；这里的参数个数指的是希望传入的形参的数量。我们通过 `getOrder(..)` 把原函数 `ajax(..)` 的参数个数从 3 个减少到了 2 个。

让我们定义一个 `partial(..)` 实用函数：

```js
function partial(fn,...presetArgs) {
	return function partiallyApplied(...laterArgs){
		return fn( ...presetArgs, ...laterArgs );
	};
}
```

**建议：**只是走马观花是不行的。请花些时间研究一下该实用函数中发生的事情。请确保你真的**理解**了。由于在接下来的文章里，我们将会一次又一次地提到该模式，所以你最好现在就适应它。

`partial(..)` 函数接收 `fn` 参数，来表示被我们偏应用实参（partially apply）的函数。接着，`fn` 形参之后，`presetArgs` 数组收集了后面传入的实参，保存起来稍后使用。

我们创建并 `return` 了一个新的内部函数（为了清晰明了，我们把它命名为`partiallyApplied(..)`），该函数中，`laterArgs` 数组收集了全部实参。

你注意到在内部函数中的 `fn` 和 `presetArgs` 引用了吗？他们是怎么如何工作的？在函数 `partial(..)` 结束运行后，内部函数为何还能访问 `fn` 和 `presetArgs` 引用？你答对了，就是因为**闭包（closure）**！内部函数 `partiallyApplied(..)` 封闭（closes over）了 `fn` 和 `presetArgs` 变量，所以无论该函数在哪里运行，在 `partial(..)` 函数运行后我们仍然可以访问这些变量。所以理解闭包是多么的重要！

当 `partiallyApplied(..)` 函数稍后在某处执行时，该函数使用被闭包作用（closed over）的 `fn` 引用来执行原函数，首先传入（被闭包作用的）`presetArgs` 数组中所有的偏应用（partial application）实参，然后再进一步传入 `laterArgs` 数组中的实参。

如果你对以上感到任何疑惑，请停下来再看一遍。相信我，随着我们进一步深入本文，你会欣然接受这个建议。

提一句，对于这类代码，函数式编程者往往喜欢使用更简短的 `=>` 箭头函数语法（请看第 2 章的 “语法” 小节），像这样：

```js
var partial =
	(fn, ...presetArgs) =>
		(...laterArgs) =>
			fn( ...presetArgs, ...laterArgs );
```

毫无疑问这更加简洁，甚至代码稀少。但我个人觉得，无论我们从数学符号的对称性上获得什么好处，都会因函数变成了匿名函数而在整体的可读性上失去更多益处。此外，由于作用域边界变得模糊，我们会更加难以辩认闭包。

不管你喜欢哪种语法实现方式，现在我们用 `partial(..)` 实用函数来制造这些之前提及的偏函数（partially-applied functions）：

```js
var getPerson = partial( ajax, "http://some.api/person" );

var getOrder = partial( ajax, "http://some.api/order" );
```

请暂停并思考一下 `getPerson(..)` 函数的外形和内在。它相当于下面这样：

```js
var getPerson = function partiallyApplied(...laterArgs) {
	return ajax( "http://some.api/person", ...laterArgs );
};
```

创建 `getOrder(..)` 函数可以依葫芦画瓢。但是 `getCurrentUser(..)` 函数又如何呢？

```js
// 版本 1
var getCurrentUser = partial(
	ajax,
	"http://some.api/person",
	{ user: CURRENT_USER_ID }
);

// 版本 2
var getCurrentUser = partial( getPerson, { user: CURRENT_USER_ID } );
```

我们可以（版本 1）直接通过指定 `url` 和 `data` 两个实参来定义 `getCurrentUser(..)` 函数，也可以（版本 2）将 `getCurrentUser(..)` 函数定义成 `getPerson(..)` 的偏应用，该偏应用仅指定一个附加的 `data` 实参。

因为版本 2 重用了已经定义好的函数，所以它在表达上更清晰一些。因此我认为它更加贴合函数式编程精神。

版本 1 和 2 分别相当于下面的代码，我们仅用这些代码来确认一下对两个函数版本内部运行机制的理解。

```js
// 版本 1
var getCurrentUser = function partiallyApplied(...laterArgs) {
	return ajax(
		"http://some.api/person",
		{ user: CURRENT_USER_ID },
		...laterArgs
	);
};

// 版本 2
var getCurrentUser = function outerPartiallyApplied(...outerLaterArgs) {
	var getPerson = function innerPartiallyApplied(...innerLaterArgs){
		return ajax( "http://some.api/person", ...innerLaterArgs );
	};

	return getPerson( { user: CURRENT_USER_ID }, ...outerLaterArgs );
}
```

再强调一下，为了确保你理解这些代码段发生了什么，请暂停并重新阅读一下它们。

**注意：**第二个版本的函数包含了一个额外的函数包装层。这看起来有些奇怪而且多余，但对于你真正要适应的函数式编程来说，这仅仅是它的冰山一角。随着本文的继续深入，我们将会把许多函数互相包装起来。记住，这就是**函数**式编程！

我们接着看另外一个偏应用的实用示例。请考虑一个 `add(..)` 函数，它接收两个实参，并取二者之和：

```js
function add(x,y) {
	return x + y;
}
```

现在，想象我们要拿到一个数字列表，并且给其中每个数字加一个确定的数值。我们将使用 JS 数组对象内置的 `map(..)` 实用函数。

```js
[1,2,3,4,5].map( function adder(val){
	return add( 3, val );
} );
// [4,5,6,7,8]
```

**注意：**如果你没见过 `map(..)` ，别担心，我们会在本书后面的部分详细介绍它。目前你只需要知道它用来循环遍历（loop over）一个数组，在遍历过程中调用函数产出新值并存到新的数组中。

因为 `add(..)` 函数签名不是 `map(..)` 函数所预期的，所以我们不直接把它传入 `map(..)` 函数里。这样一来，偏应用就有了用武之地：我们可以调整 `add(..)` 函数签名，以符合 `map(..)` 函数的预期。

```js
[1,2,3,4,5].map( partial( add, 3 ) );
// [4,5,6,7,8]
```

### `bind(..)`

JavaScript 有一个内建的 `bind(..)` 实用函数，任何函数都可以使用它。该函数有两个功能：预设 `this` 关键字的上下文，以及偏应用实参。

我认为将这两个功能混合进一个实用函数是极其糟糕的决定。有时你不想关心 `this` 的绑定，而只是要偏应用实参。我本人基本上从不会同时需要这两个功能。

对于下面的方案，你通常要传 `null` 给用来绑定 `this` 的实参（第一个实参），而它是一个可以忽略的占位符。因此，这个方案非常糟糕。

请看:

```js
var getPerson = ajax.bind( null, "http://some.api/person" );
```

那个 `null` 只会给我带来无尽的烦恼。

### 将实参顺序颠倒

回想我们之前调用 Ajax 函数的方式：`ajax( url, data, cb )`。如果要偏应用 `cb` 而稍后再指定 `data` 和 `url` 参数，我们应该怎么做呢？我们可以创建一个可以颠倒实参顺序的实用函数，用来包装原函数。

```js
function reverseArgs(fn) {
	return function argsReversed(...args){
		return fn( ...args.reverse() );
	};
}

// ES6 箭头函数形式
var reverseArgs =
	fn =>
		(...args) =>
			fn( ...args.reverse() );
```

现在可以颠倒 `ajax(..)` 实参的顺序了，接下来，我们不再从左边开始，而是从右侧开始偏应用实参。为了恢复期望的实参顺序，接着我们又将偏应用实参后的函数颠倒一下实参顺序：

```js
var cache = {};

var cacheResult = reverseArgs(
	partial( reverseArgs( ajax ), function onResult(obj){
		cache[obj.id] = obj;
	} )
);

// 处理后:
cacheResult( "http://some.api/person", { user: CURRENT_USER_ID } );
```

好，我们来定义一个从右边开始偏应用实参（译注：以下简称右偏应用实参）的 `partialRight(..)` 实用函数。我们将运用和上面相同的技巧于该函数中：

```js
function partialRight( fn, ...presetArgs ) {
	return reverseArgs(
		partial( reverseArgs( fn ), ...presetArgs.reverse() )
	);
}

var cacheResult = partialRight( ajax, function onResult(obj){
	cache[obj.id] = obj;
});

// 处理后:
cacheResult( "http://some.api/person", { user: CURRENT_USER_ID } );
```

这个 `partialRight(..)` 函数的实现方案不能保证让一个特定的形参接受特定的被偏应用的值；它只能确保将被这些值（一个或几个）当作原函数最右边的实参（一个或几个）传入。

举个例子:

```js
function foo(x,y,z) {
	var rest = [].slice.call( arguments, 3 );
	console.log( x, y, z, rest );
}

var f = partialRight( foo, "z:last" );

f( 1, 2 );			// 1 2 "z:last" []

f( 1 );				// 1 "z:last" undefined []

f( 1, 2, 3 );		// 1 2 3 ["z:last"]

f( 1, 2, 3, 4 );	// 1 2 3 [4,"z:last"]
```

只有在传两个实参（匹配到 `x` 和 `y` 形参）调用 `f(..)` 函数时，`"z:last"` 这个值才能被赋给函数的形参 `z`。在其他的例子里，不管左边有多少个实参，`"z:last"` 都被传给最右的实参。

## 一次传一个

我们来看一个跟偏应用类似的技术，该技术将一个期望接收多个实参的函数拆解成连续的链式函数（chained functions），每个链式函数接受单一实参（实参个数：1）并返回另一个接收下一个实参的函数。

这就是柯里化（currying）技术。

首先，想象我们已创建了一个 `ajax(..)` 的柯里化版本。我们这样使用它：

```js
curriedAjax( "http://some.api/person" )
	( { user: CURRENT_USER_ID } )
		( function foundUser(user){ /* .. */ } );
```

我们将三次调用分别拆解开来，这也许有助于我们理解整个过程：

```js
var personFetcher = curriedAjax( "http://some.api/person" );

var getCurrentUser = personFetcher( { user: CURRENT_USER_ID } );

getCurrentUser( function foundUser(user){ /* .. */ } );
```

该 `curriedAjax(..)` 函数在每次调用中，一次只接收一个实参，而不是一次性接收所有实参（像 `ajax(..)` 那样），也不是先传部分实参再传剩余部分实参（借助 `partial(..)` 函数）。

柯里化和偏应用相似，每个类似偏应用的连续柯里化调用都把另一个实参应用到原函数，一直到所有实参传递完毕。

不同之处在于，`curriedAjax(..)` 函数会明确地返回一个期望**只接收下一个实参** `data` 的函数（我们把它叫做 `curriedGetPerson(..)`），而不是那个能接收所有剩余实参的函数(像此前的 `getPerson(..)` 函数) 。

如果一个原函数期望接收 5 个实参，这个函数的柯里化形式只会接收第一个实参，并且返回一个用来接收第二个参数的函数。而这个被返回的函数又只接收第二个参数，并且返回一个接收第三个参数的函数。依此类推。

由此而知，柯里化将一个多参数（higher-arity）函数拆解为一系列的单元链式函数。

如何定义一个用来柯里化的实用函数呢？我们将要用到第 2 章中的一些技巧。

```js
function curry(fn,arity = fn.length) {
	return (function nextCurried(prevArgs){
		return function curried(nextArg){
			var args = prevArgs.concat( [nextArg] );

			if (args.length >= arity) {
				return fn( ...args );
			}
			else {
				return nextCurried( args );
			}
		};
	})( [] );
}
```

ES6 箭头函数版本：

```js
var curry =
	(fn, arity = fn.length, nextCurried) =>
		(nextCurried = prevArgs =>
			nextArg => {
				var args = prevArgs.concat( [nextArg] );

				if (args.length >= arity) {
					return fn( ...args );
				}
				else {
					return nextCurried( args );
				}
			}
		)( [] );
```

此处的实现方式是把空数组 `[]` 当作 `prevArgs` 的初始实参集合，并且将每次接收到的 `nextArg` 同 `prevArgs` 连接成 `args` 数组。当 `args.length` 小于 `arity`（原函数 `fn(..)` 被定义和期望的形参数量）时，返回另一个 `curried(..)` 函数（译注：这里指代 `nextCurried(..)` 返回的函数）用来接收下一个 `nextArg` 实参，与此同时将 `args` 实参集合作为唯一的 `prevArgs` 参数传入 `nextCurried(..)` 函数。一旦我们收集了足够长度的 `args` 数组，就用这些实参触发原函数 `fn(..)`。

默认地，我们的实现方案基于下面的条件：在拿到原函数期望的全部实参之前，我们能够通过检查将要被柯里化的函数的 `length` 属性来得知柯里化需要迭代多少次。

假如你将该版本的 `curry(..)` 函数用在一个 `length` 属性不明确的函数上 —— 函数的形参声明包含默认形参值、形参解构，或者它是可变参数函数，用 `...args` 当形参；参考第 2 章 —— 你将要传入 `arity` 参数（作为 `curry(..)` 的第二个形参）来确保 `curry(..)` 函数的正常运行。

我们用 `curry(..)` 函数来实现此前的 `ajax(..)` 例子：

```js
var curriedAjax = curry( ajax );

var personFetcher = curriedAjax( "http://some.api/person" );

var getCurrentUser = personFetcher( { user: CURRENT_USER_ID } );

getCurrentUser( function foundUser(user){ /* .. */ } );
```

如上，我们每次函数调用都会新增一个实参，最终给原函数 `ajax(..)` 使用，直到收齐三个实参并执行 `ajax(..)` 函数为止。

还记得前面讲到为数值列表的每个值加 `3` 的那个例子吗？回顾一下，由于柯里化是和偏应用相似的，所以我们可以用几乎相同的方式以柯里化来完成那个例子。

```js
[1,2,3,4,5].map( curry( add )( 3 ) );
// [4,5,6,7,8]
```

`partial(add,3)` 和 `curry(add)(3)` 两者有什么不同呢？为什么你会选 `curry(..)` 而不是偏函数呢？当你先得知 `add(..)` 是将要被调整的函数，但如果这个时候并不能确定 `3` 这个值，柯里化可能会起作用：

```js
var adder = curry( add );

// later
[1,2,3,4,5].map( adder( 3 ) );
// [4,5,6,7,8]
```

让我们来看看另一个有关数字的例子，这次我们拿一个列表的数字做加法：

```js
function sum(...args) {
	var sum = 0;
	for (let i = 0; i < args.length; i++) {
		sum += args[i];
	}
	return sum;
}

sum( 1, 2, 3, 4, 5 );						// 15

// 好，我们看看用柯里化怎么做:
// (5 用来指定需要链式调用的次数)
var curriedSum = curry( sum, 5 );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );		// 15
```

这里柯里化的好处是，每次函数调用传入一个实参，并生成另一个特定性更强的函数，之后我们可以在程序中获取并使用**那个**新函数。而偏应用则是预先指定所有将被偏应用的实参，产出一个等待接收剩下所有实参的函数。

如果想用偏应用来每次指定一个形参，你得在每个函数中逐次调用 `partialApply(..)` 函数。而被柯里化的函数可以自动完成这个工作，这让一次单独传递一个参数变得更加符合人机工程学。

在 JavaScript 中，柯里化和偏应用都使用闭包来保存实参，直到收齐所有实参后我们再执行原函数。

### 柯里化和偏应用有什么用？

无论是柯里化风格（`sum(1)(2)(3)`）还是偏应用风格（`partial(sum,1,2)(3)`），它们的签名比普通函数签名奇怪得多。那么，在适应函数式编程的时候，**我们为什么要这么做呢**？答案有几个方面。

首先是显而易见的理由，使用柯里化和偏应用可以将指定分离实参的时机和地方独立开来（遍及代码的每一处），而传统函数调用则需要预先确定所有实参。如果你在代码某一处只获取了部分实参，然后在另一处确定另一部分实参，这个时候柯里化和偏应用就能派上用场。

另一个最能体现柯里化应用的的是，当函数只有一个形参时，我们能够比较容易地组合它们。因此，如果一个函数最终需要三个实参，那么它被柯里化以后会变成需要三次调用，每次调用需要一个实参的函数。当我们组合函数时，这种单元函数的形式会让我们处理起来更简单。我们将在后面继续探讨这个话题。

### 如何柯里化多个实参？

到目前为止，我相信我给出的是我们能在 JavaScript 中能得到的，最精髓的柯里化定义和实现方式。

具体来说，如果简单看下柯里化在 Haskell 语言中的应用，我们会发现一个函数总是在一次柯里化调用中接收多个实参 —— 而不是接收一个包含多个值的元组（tuple，类似我们的数组）实参。

在 Haskell 中的示例：

```
foo 1 2 3
```

该示例调用了 `foo` 函数，并且根据传入的三个值 `1`、`2` 和 `3` 得到了结果。但是在 Haskell 中，函数会自动被柯里化，这意味着我们传入函数的值都分别传入了单独的柯里化调用。在 JS 中看起来则会是这样：`foo(1)(2)(3)`。这和我此前讲过的 `curry(..)` 风格如出一辙。

**注意：** 在 Haskell 中，`foo (1,2,3)` 不是把三个值当作单独的实参一次性传入函数，而是把它们包含在一个元组（类似 JS 数组）中作为单独实参传入函数。为了正常运行，我们需要改变 `foo` 函数来处理作为实参的元组。据我所知，在 Haskell 中我们没有办法在一次函数调用中将全部三个实参独立地传入，而需要柯里化调用每个函数。诚然，多次调用对于 Haskell 开发者来说是透明的，但对 JS 开发者来说，这在语法上更加一目了然。

基于以上原因，我认为此前展示的 `curry(..)` 函数是一个对 Haskell 柯里化的可靠改编，我把它叫做 “严格柯里化”。

然而，我们需要注意，大多数流行的 JavaScript 函数式编程库都使用了一种并不严格的柯里化（loose currying）定义。

具体来说，往往 JS 柯里化实用函数会允许你在每次柯里化调用中指定多个实参。回顾一下之前提到的 `sum(..)` 示例，松散柯里化应用会是下面这样：

```js
var curriedSum = looseCurry( sum, 5 );

curriedSum( 1 )( 2, 3 )( 4, 5 );			// 15
```

可以看到，语法上我们节省了`()`的使用，并且把五次函数调用减少成三次，间接提高了性能。除此之外，使用 `looseCurry(..)` 函数的结果也和之前更加狭义的 `curry(..)` 函数一样。我猜便利性和性能因素是众框架允许多实参柯里化的原因。这看起来更像是品味问题。

**注意：**松散柯里化**允许**你传入超过形参数量（arity，原函数确认或指定的形参数量）的实参。如果你将函数的参数设计成可配的或变化的，那么松散柯里化将会有利于你。例如，如果你将要柯里化的函数接收 5 个实参，松散柯里化依然允许传入超过 5 个的实参（`curriedSum(1)(2,3,4)(5,6)`），而严格柯里化就不支持 `curriedSum(1)(2)(3)(4)(5)(6)`。

我们可以将之前的柯里化实现方式调整一下，使其适应这种常见的更松散的定义：

```js
function looseCurry(fn,arity = fn.length) {
	return (function nextCurried(prevArgs){
		return function curried(...nextArgs){
			var args = prevArgs.concat( nextArgs );

			if (args.length >= arity) {
				return fn( ...args );
			}
			else {
				return nextCurried( args );
			}
		};
	})( [] );
}
```

现在每个柯里化调用可以接收一个或多个实参了（收集在 `nextArgs` 数组中）。至于这个实用函数的 ES6 箭头函数版本，我们就留作一个小练习，有兴趣的读者可以模仿之前 `curry(..)` 函数的来完成。

### 反柯里化

你也会遇到这种情况：拿到一个柯里化后的函数，却想要它柯里化之前的版本 —— 这本质上就是想将类似 `f(1)(2)(3)` 的函数变回类似 `g(1,2,3)` 的函数。

不出意料的话，处理这个需求的标准实用函数通常被叫作 `uncurry(..)`。下面是简陋的实现方式：

```js
function uncurry(fn) {
	return function uncurried(...args){
		var ret = fn;

		for (let i = 0; i < args.length; i++) {
			ret = ret( args[i] );
		}

		return ret;
	};
}

// ES6 箭头函数形式
var uncurry =
	fn =>
		(...args) => {
			var ret = fn;

			for (let i = 0; i < args.length; i++) {
				ret = ret( args[i] );
			}

			return ret;
		};
```

**警告：**请不要以为 `uncurry(curry(f))` 和 `f` 函数的行为完全一样。虽然在某些库中，反柯里化使函数变成和原函数（译注：这里的原函数指柯里化之前的函数）类似的函数，但是凡事皆有例外，我们这里就有一个例外。如果你传入原函数期望数量的实参，那么在反柯里化后，函数的行为（大多数情况下）和原函数相同。然而，如果你少传了实参，就会得到一个仍然在等待传入更多实参的部分柯里化函数。我们在下面的代码中说明这个怪异行为。

```js
function sum(...args) {
	var sum = 0;
	for (let i = 0; i < args.length; i++) {
		sum += args[i];
	}
	return sum;
}

var curriedSum = curry( sum, 5 );
var uncurriedSum = uncurry( curriedSum );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );		// 15

uncurriedSum( 1, 2, 3, 4, 5 );				// 15
uncurriedSum( 1, 2, 3 )( 4 )( 5 );			// 15
```

`uncurry()` 函数最为常见的作用对象很可能并不是人为生成的柯里化函数（例如上文所示），而是某些操作所产生的已经被柯里化了的结果函数。我们将在本章后面关于 “无形参风格” 的讨论中阐述这种应用场景。

## 只要一个实参 

设想你向一个实用函数传入一个函数，而这个实用函数会把多个实参传入函数，但可能你只希望你的函数接收单一实参。如果你有个类似我们前面提到被松散柯里化的函数，它能接收多个实参，但你却想让它接收单一实参。那么这就是我想说的情况。

我们可以设计一个简单的实用函数，它包装一个函数调用，确保被包装的函数只接收一个实参。既然实际上我们是强制把一个函数处理成单元函数（unary），那我们索性就这样命名实用函数：

```js
function unary(fn) {
	return function onlyOneArg(arg){
		return fn( arg );
	};
}

// ES6 箭头函数形式
var unary =
	fn =>
		arg =>
			fn( arg );
```

我们此前已经和 `map(..)` 函数打过照面了。它调用传入其中的 mapping 函数时会传入三个实参：`value`、`index` 和 `list`。如果你希望你传入 `map(..)` 的 mapping 函数只接受一个参数，比如 `value`，你可以使用 `unary(..)` 函数来操作：

```js
function unary(fn) {
	return function onlyOneArg(arg){
		return fn( arg );
	};
}

var adder = looseCurry( sum, 2 );

// 出问题了：
[1,2,3,4,5].map( adder( 3 ) );
// ["41,2,3,4,5", "61,2,3,4,5", "81,2,3,4,5", "101, ...

// 用 `unary(..)` 修复后:
[1,2,3,4,5].map( unary( adder( 3 ) ) );
// [4,5,6,7,8]
```

另一种常用的 `unary(..)` 函数调用示例：

```js
["1","2","3"].map( parseFloat );
// [1,2,3]

["1","2","3"].map( parseInt );
// [1,NaN,NaN]

["1","2","3"].map( unary( parseInt ) );
// [1,2,3]
```

对于 `parseInt(str,radix)` 这个函数调用，如果 `map(..)` 函数调用它时在它的第二个实参位置传入 `index`，那么毫无疑问 `parseInt(..)` 会将 `index` 理解为 `radix` 参数，这是我们不希望发生的。而 `unary(..)` 函数创建了一个只接收第一个传入实参，忽略其他实参的新函数，这就意味着传入 `index` 不再会被误解为 `radix` 参数。

### 传一个返回一个

说到只传一个实参的函数，在函数式编程工具库中有另一种通用的基础函数：该函数接收一个实参，然后什么都不做，原封不动地返回实参值。

```js
function identity(v) {
	return v;
}

// ES6 箭头函数形式
var identity =
	v =>
		v;
```

看起来这个实用函数简单到了无处可用的地步。但即使是简单的函数在函数式编程的世界里也能发挥作用。就像演艺圈有句谚语：没有小角色，只有小演员。

举个例子，想象一下你要用正则表达式拆分（split up）一个字符串，但输出的数组中可能包含一些空值。我们可以使用 `filter(..)` 数组方法（下文会详细说到这个方法）来筛除空值，而我们将 `identity(..)` 函数作为 `filter(..)` 的断言：

```js
var words = "   Now is the time for all...  ".split( /\s|\b/ );
words;
// ["","Now","is","the","time","for","all","...",""]

words.filter( identity );
// ["Now","is","the","time","for","all","..."]
```

既然 `identity(..)` 会简单地返回传入的值，而 JS 会将每个值强制转换为 `true` 或 `false`，这样我们就能在最终的数组里对每个值进行保存或排除。

**小贴士：**像这个例子一样，另外一个能被用作断言的单实参函数是 JS 自有的 `Boolean(..)` 方法，该方法会强制把传入值转为 `true` 或 `false`。

另一个使用 `identity(..)` 的示例就是将其作为替代一个转换函数（译注：transformation，这里指的是对传入值进行修改或调整，返回新值的函数）的默认函数：

```js
function output(msg,formatFn = identity) {
	msg = formatFn( msg );
	console.log( msg );
}

function upper(txt) {
	return txt.toUpperCase();
}

output( "Hello World", upper );		// HELLO WORLD
output( "Hello World" );			// Hello World
```

如果不给 `output(..)` 函数的 `formatFn` 参数设置默认值，我们可以叫出老朋友 `partialRight(..)` 函数：

```js
var specialOutput = partialRight( output, upper );
var simpleOutput = partialRight( output, identity );

specialOutput( "Hello World" );		// HELLO WORLD
simpleOutput( "Hello World" );		// Hello World
```

你也可能会看到 `identity(..)` 被当作 `map(..)` 函数调用的默认转换函数，或者作为某个函数数组的 `reduce(..)` 函数的初始值。我们将会在第 8 章中提到这两个实用函数。

### 恒定参数

Certain API 禁止直接给方法传值，而要求我们传入一个函数，就算这个函数只是返回一个值。JS Promise 中的 `then(..)` 方法就是一个 Certain API。很多人声称 ES6 箭头函数可以当作这个问题的 “解决方案”。但我这有一个函数式编程实用函数可以完美胜任该任务：

```js
function constant(v) {
	return function value(){
		return v;
	};
}

// or the ES6 => form
var constant =
	v =>
		() =>
			v;
```

这个微小而简洁的实用函数可以解决我们关于 `then(..)` 的烦恼：

```js
p1.then( foo ).then( () => p2 ).then( bar );

// 对比：

p1.then( foo ).then( constant( p2 ) ).then( bar );
```

**警告：**尽管使用 `() => p2` 箭头函数的版本比使用 `constant(p2)` 的版本更简短，但我建议你忍住别用前者。该箭头函数返回了一个来自外作用域的值，这和 函数式编程的理念有些矛盾。我们将会在后面第 5 章的 “减少副作用” 小节中提到这种行为带来的陷阱。

## 扩展在参数中的妙用

在第 2 章中，我们简要地讲到了形参数组解构。回顾一下该示例：

```js
function foo( [x,y,...args] ) {
	// ..
}

foo( [1,2,3] );
```

在 `foo(..)` 函数的形参列表中，我们期望接收单一数组实参，我们要把这个数组拆解 —— 或者更贴切地说，扩展（spread out）—— 成独立的实参 `x` 和 `y`。除了头两个位置以外的参数值我们都会通过 `...` 操作将它们收集在 `args` 数组中。

当函数必须接收一个数组，而你却想把数组内容当成单独形参来处理的时候，这个技巧十分有用。

然而，有的时候，你无法改变原函数的定义，但想使用形参数组解构。举个例子，请思考下面的函数：

```js
function foo(x,y) {
	console.log( x + y );
}

function bar(fn) {
	fn( [ 3, 9 ] );
}

bar( foo );			// 失败
```

你注意到为什么 `bar(foo)` 函数失败了吗？

我们将 `[3,9]` 数组作为单一值传入 `fn(..)` 函数，但 `foo(..)` 期望接收单独的 `x` 和 `y` 形参。如果我们可以把 `foo(..)` 的函数声明改变成 `function foo([x,y]) { ..` 那就好办了。或者，我们可以改变 `bar(..)` 函数的行为，把调用改成 `fn(...[3,9])`，这样就能将 `3` 和 `9` 分别传入 `foo(..)` 函数了。

假设有两个在此方法上互不兼容的函数，而且由于各种原因你无法改变它们的声明和定义。那么你该如何一并使用它们呢？

为了调整一个函数，让它能把接收的单一数组扩展成各自独立的实参，我们可以定义一个辅助函数：

```js
function spreadArgs(fn) {
	return function spreadFn(argsArr) {
		return fn( ...argsArr );
	};
}

// ES6 箭头函数的形式：
var spreadArgs =
	fn =>
		argsArr =>
			fn( ...argsArr );
```

**注意：**我把这个辅助函数叫做 `spreadArgs(..)`，但一些库，比如 Ramda，经常把它叫做 `apply(..)`。

现在我们可以使用 `spreadArgs(..)` 来调整 `foo(..)` 函数，使其作为一个合适的输入参数并正常地工作：

```js
bar( spreadArgs( foo ) );			// 12
```

相信我，虽然我不能讲清楚这些问题出现的原因，但它们一定会出现的。本质上，`spreadArgs(..)` 函数使我们能够定义一个借助数组 `return` 多个值的函数，不过，它让这些值仍然能分别作为其他函数的输入参数来处理。

一个函数的输出作为另外一个函数的输入被称作组合（composition），我们将在第四章详细讨论这个话题。

尽管我们在谈论 `spreadArgs(..)` 实用函数，但我们也可以定义一下实现相反功能的实用函数：

```js
function gatherArgs(fn) {
	return function gatheredFn(...argsArr) {
		return fn( argsArr );
	};
}

// ES6 箭头函数形式
var gatherArgs =
	fn =>
		(...argsArr) =>
			fn( argsArr );
```

**注意：** 在 Ramda 中，该实用函数被称作 `unapply(..)`，是与 `apply(..)` 功能相反的函数。我认为术语 “扩展（spread）” 和 “聚集（gather）” 可以把这两个函数发生的事情解释得更好一些。

因为有时我们可能要调整一个函数，解构其数组形参，使其成为另一个分别接收单独实参的函数，所以我们可以通过使用 `gatherArgs(..)` 实用函数来将单独的实参聚集到一个数组中。我们将在第 8 章中细说 `reduce(..)` 函数，这里我们简要说一下：它重复调用传入的 reducer 函数，其中 reducer 函数有两个形参，现在我们可以将这两个形参**聚集**起来：

```js
function combineFirstTwo([ v1, v2 ]) {
	return v1 + v2;
}

[1,2,3,4,5].reduce( gatherArgs( combineFirstTwo ) );
// 15
```

## 参数顺序的那些事儿

对于多形参函数的柯里化和偏应用，我们不得不通过许多令人懊恼的技巧来修正这些形参的顺序。有时我们把一个函数的形参顺序定义成柯里化需求的形参顺序，但这种顺序没有兼容性，我们不得不绞尽脑汁来重新调整它。

让人沮丧的可不仅是我们需要使用实用函数来委曲求全，在此之外，这种做法还会导致我们的代码被无关代码混淆。这种东西就像碎纸片，这一片那一片的，而不是一整个突出问题，但这些问题的细碎丝毫不会减少它们带来的苦恼。

难道就没有能让我们从修正参数顺序这件事里解脱出来的方法吗！？

在第 2 章里，我们讲到了命名实参（named-argument）解构模式。回顾一下：

```js
function foo( {x,y} = {} ) {
	console.log( x, y );
}

foo( {
	y: 3
} );					// undefined 3
```

我们将 `foo(..)` 函数的第一个形参 —— 它被期望是一个对象 —— 解构成单独的形参 `x` 和 `y`。接着在调用时传入一个对象实参，并且提供函数期望的属性，这样就可以把 “命名实参” 映射到相应形参上。

命名实参主要的好处就是不用再纠结实参传入的顺序，因此提高了可读性。我们可以发掘一下看看是否能设计一个等效的实用函数来处理对象属性，以此提高柯里化和偏应用的可读性：

```js
function partialProps(fn,presetArgsObj) {
	return function partiallyApplied(laterArgsObj){
		return fn( Object.assign( {}, presetArgsObj, laterArgsObj ) );
	};
}

function curryProps(fn,arity = 1) {
	return (function nextCurried(prevArgsObj){
		return function curried(nextArgObj = {}){
			var [key] = Object.keys( nextArgObj );
			var allArgsObj = Object.assign( {}, prevArgsObj, { [key]: nextArgObj[key] } );

			if (Object.keys( allArgsObj ).length >= arity) {
				return fn( allArgsObj );
			}
			else {
				return nextCurried( allArgsObj );
			}
		};
	})( {} );
}
```

我们甚至不需要设计一个 `partialPropsRight(..)` 函数了，因为我们根本不需要考虑属性的映射顺序，通过命名来映射形参完全解决了我们有关于顺序的烦恼！

我们这样使用这些使用函数：

```js
function foo({ x, y, z } = {}) {
	console.log( `x:${x} y:${y} z:${z}` );
}

var f1 = curryProps( foo, 3 );
var f2 = partialProps( foo, { y: 2 } );

f1( {y: 2} )( {x: 1} )( {z: 3} );
// x:1 y:2 z:3

f2( { z: 3, x: 1 } );
// x:1 y:2 z:3
```

我们不用再为参数顺序而烦恼了！现在，我们可以指定我们想传入的实参，而不用管它们的顺序如何。再也不需要类似 `reverseArgs(..)` 的函数或其它妥协了。赞！

### 属性扩展

不幸的是，只有在我们可以掌控 `foo(..)` 的函数签名，并且可以定义该函数的行为，使其解构第一个参数的时候，以上技术才能起作用。如果一个函数，其形参是各自独立的（没有经过形参解构），而且不能改变它的函数签名，那我们应该如何运用这个技术呢？

```js
function bar(x,y,z) {
	console.log( `x:${x} y:${y} z:${z}` );
}
```

就像之前的 `spreadArgs(..)` 实用函数一样，我们也可以定义一个 `spreadArgProps(..)` 辅助函数，它接收对象实参的 `key: value` 键值对，并将其 “扩展” 成独立实参。

不过，我们需要注意某些异常的地方。我们使用 `spreadArgs(..)` 函数处理数组实参时，参数的顺序是明确的。然而，对象属性的顺序是不太明确且不可靠的。取决于不同对象的创建方式和属性设置方式，我们无法完全确认对象会产生什么顺序的属性枚举。

针对这个问题，我们定义的实用函数需要让你能够指定函数期望的实参顺序（比如属性枚举的顺序）。我们可以传入一个类似 `["x","y","z"]` 的数组，通知实用函数基于该数组的顺序来获取对象实参的属性值。

这着实不错，但还是有点瑕疵，就算是最简单的函数，我们也**免不了**为其增添一个由属性名构成的数组。难道我们就没有一种可以探知函数形参顺序的技巧吗？哪怕给一个普通而简单的例子？还真有！

JavaScript 的函数对象上有一个 `.toString()` 方法，它返回函数代码的字符串形式，其中包括函数声明的签名。先忽略其正则表达式分析技巧，我们可以通过解析函数字符串来获取每个单独的命名形参。虽然这段代码看起来有些粗暴，但它足以满足我们的需求：

```js
function spreadArgProps(
	fn,
	propOrder =
		fn.toString()
		.replace( /^(?:(?:function.*\(([^]*?)\))|(?:([^\(\)]+?)\s*=>)|(?:\(([^]*?)\)\s*=>))[^]+$/, "$1$2$3" )
		.split( /\s*,\s*/ )
		.map( v => v.replace( /[=\s].*$/, "" ) )
) {
	return function spreadFn(argsObj) {
		return fn( ...propOrder.map( k => argsObj[k] ) );
	};
}
```

**注意：**该实用函数的参数解析逻辑并非无懈可击，使用正则来解析代码这个前提就已经很不靠谱了！但处理一般情况是我们的唯一目标，从这点来看这个实用函数还是恰到好处的。我们需要的只是对简单形参（包括带默认值的形参）函数的形参顺序做一个恰当的默认检测。例如，我们的实用函数不需要把复杂的解构形参给解析出来，因为无论如何我们不太可能对拥有这种复杂形参的函数使用 `spreadArgProps()` 函数。因此该逻辑能搞定 80% 的需求，它允许我们在其它不能正确解析复杂函数签名的情况下覆盖 `propOrder` 数组形参。这是本书尽可能寻找的一种实用性平衡。

让我们看看 `spreadArgProps(..)` 实用函数是怎么用的：

```js
function bar(x,y,z) {
	console.log( `x:${x} y:${y} z:${z}` );
}

var f3 = curryProps( spreadArgProps( bar ), 3 );
var f4 = partialProps( spreadArgProps( bar ), { y: 2 } );

f3( {y: 2} )( {x: 1} )( {z: 3} );
// x:1 y:2 z:3

f4( { z: 3, x: 1 } );
// x:1 y:2 z:3
```

提个醒：本文中呈现的对象形参（object parameters）和命名实参（named arguments）模式，通过减少由调整实参顺序带来的干扰，明显地提高了代码的可读性，不过据我所知，没有哪个主流的函数式编程库使用该方案。所以你会看到该做法与大多数 JavaScript 函数式编程很不一样.

此外，使用在这种风格下定义的函数要求你知道每个实参的名字。你必须记住：“这个函数形参叫作 ‘fn’ ”，而不是只记得：“噢，把这个函数作为第一个实参传进去”。

请小心地权衡它们。

## 无形参风格

在函数式编程的世界中，有一种流行的代码风格，其目的是通过移除不必要的形参-实参映射来减少视觉上的干扰。这种风格的正式名称为 “隐性编程（tacit programming）”，一般则称作 “无形参（point-free）” 风格。术语 “point” 在这里指的是函数形参。

**警告：** 且慢，先说明我们这次的讨论是一个有边界的提议，我不建议你在函数式编程的代码里不惜代价地滥用无形参风格。该技术是用于在适当情况下提升可读性。但你完全可能像滥用软件开发里大多数东西一样滥用它。如果你由于必须迁移到无参数风格而让代码难以理解，请打住。你不会因此获得小红花，因为你用看似聪明但晦涩难懂的方式抹除形参这个点的同时，还抹除了代码的重点。

我们从一个简单的例子开始：

```js
function double(x) {
	return x * 2;
}

[1,2,3,4,5].map( function mapper(v){
	return double( v );
} );
// [2,4,6,8,10]
```

可以看到 `mapper(..)` 函数和 `double(..)` 函数有相同（或相互兼容）的函数签名。形参（也就是所谓的 “point“）`v` 可以直接映射到 `double(..)` 函数调用里相应的实参上。这样，`mapper(..)` 函数包装层是非必需的。我们可以将其简化为无形参风格：

```js
function double(x) {
	return x * 2;
}

[1,2,3,4,5].map( double );
// [2,4,6,8,10]
```

回顾之前的一个例子：

```js
["1","2","3"].map( function mapper(v){
	return parseInt( v );
} );
// [1,2,3]
```

该例中，`mapper(..)` 实际上起着重要作用，它排除了 `map(..)` 函数传入的 `index` 实参，因为如果不这么做的话，`parseInt(..)` 函数会错把 `index` 当作 `radix` 来进行整数解析。该例子中我们可以借助 `unary(..)` 函数：

```js
["1","2","3"].map( unary( parseInt ) );
// [1,2,3]
```

使用无形参风格的关键，是找到你代码中，有哪些地方的函数直接将其形参作为内部函数调用的实参。以上提到的两个例子中，`mapper(..)` 函数拿到形参 `v` 单独传入了另一个函数调用。我们可以借助 `unary(..)` 函数将提取形参的逻辑层替换成无参数形式表达式。

**警告：**你可能跟我一样，已经尝试着使用 `map(partialRight(parseInt,10))` 来将 `10` 右偏应用为 `parseInt(..)` 的 `radix` 实参。然而，就像我们之前看到的那样，`partialRight(..)` 仅仅保证将 `10` 当作最后一个实参传入原函数，而不是将其指定为第二个实参。因为 `map(..)` 函数本身会将 3 个实参（`value`、`index` 和 `arr`）传入它的映射函数，所以 `10` 就会被当成第四个实参传入 `parseInt(..)` 函数，而这个函数只会对头两个实参作出反应。

来看另一个例子：

```js

// 将 `console.log` 当成一个函数使用
// 便于避免潜在的绑定问题

function output(txt) {
	console.log( txt );
}

function printIf( predicate, msg ) {
	if (predicate( msg )) {
		output( msg );
	}
}

function isShortEnough(str) {
	return str.length <= 5;
}

var msg1 = "Hello";
var msg2 = msg1 + " World";

printIf( isShortEnough, msg1 );			// Hello
printIf( isShortEnough, msg2 );
```

现在，我们要求当信息足够长时，将它打印出来，换而言之，我们需要一个 `!isShortEnough(..)` 断言。你可能会首先想到：

```js
function isLongEnough(str) {
	return !isShortEnough( str );
}

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );			// Hello World
```

这太简单了...但现在我们的重点来了！你看到了 `str` 形参是如何传递的吗？我们能否不通过重新实现 `str.length` 的检查逻辑，而重构代码并使其变成无形参风格呢？

我们定义一个 `not(..)` 取反辅助函数（在函数式编程库中又被称作 `complement(..)`）：

```js
function not(predicate) {
	return function negated(...args){
		return !predicate( ...args );
	};
}

// ES6 箭头函数形式
var not =
	predicate =>
		(...args) =>
			!predicate( ...args );
```

接着，我们使用 `not(..)` 函数来定义无形参的 `isLongEnough(..)` 函数：

```js
var isLongEnough = not( isShortEnough );

printIf( isLongEnough, msg2 );			// Hello World
```

目前为止已经不错了，但**还能**更进一步。我们实际上可以将 `printIf(..)` 函数本身重构成无形参风格。

我们可以用 `when(..)` 实用函数来表示 `if` 条件句：

```js
function when(predicate,fn) {
	return function conditional(...args){
		if (predicate( ...args )) {
			return fn( ...args );
		}
	};
}

// ES6 箭头函数形式
var when =
	(predicate,fn) =>
		(...args) =>
			predicate( ...args ) ? fn( ...args ) : undefined;
```

我们把本章前面讲到的另一些辅助函数和 `when(..)` 函数结合起来搞定无形参风格的 `printIf(..)` 函数：

```js
var printIf = uncurry( rightPartial( when, output ) );
```

我们是这么做的：将 `output` 方法右偏应用为 `when(..)` 函数的第二个（`fn` 形参）实参，这样我们得到了一个仍然期望接收第一个实参（`predicate` 形参）的函数。当**该函数**被调用时，会产生另一个期望接收（译注：需要被打印的）信息字符串的函数，看起来就是这样：`fn(predicate)(str)`。

多个（两个）链式函数的调用看起来很挫，就像被柯里化的函数。于是我们用 `uncurry(..)` 函数处理它，得到一个期望接收 `str` 和 `predicate` 两个实参的函数，这样该函数的签名就和 `printIf(predicate,str)` 原函数一样了。

我们把整个例子复盘一下（假设我们本章已经讲解的实用函数都在这里了）：

```js
function output(msg) {
	console.log( msg );
}

function isShortEnough(str) {
	return str.length <= 5;
}

var isLongEnough = not( isShortEnough );

var printIf = uncurry( partialRight( when, output ) );

var msg1 = "Hello";
var msg2 = msg1 + " World";

printIf( isShortEnough, msg1 );			// Hello
printIf( isShortEnough, msg2 );

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );			// Hello World
```

但愿无形参风格编程的函数式编程实践逐渐变得更有意义。你仍然可以通过大量实践来训练自己，让自己接受这种风格。**再次提醒，请三思而后行**，掂量一下是否值得使用无形参风格编程，以及使用到什么程度会益于提高代码的可读性。

有形参还是无形参，你怎么选？

**注意：** 还有什么无形参风格编程的实践呢？我们将在第 4 章的 “回顾形参” 小节里，站在新学习的组合函数知识之上来回顾这个技术。

## 总结

偏应用是用来减少函数的参数数量 —— 一个函数期望接收的实参数量 —— 的技术，它减少参数数量的方式是创建一个预设了部分实参的新函数。

柯里化是偏应用的一种特殊形式，其参数数量降低为 1，这种形式包含一串连续的链式函数调用，每个调用接收一个实参。当这些链式调用指定了所有实参时，原函数就会拿到收集好的实参并执行。你同样可以将柯里化还原。

其它类似 `unary(..)`、`identity(..)` 以及 `constant(..)` 的重要函数操作，是函数式编程基础工具库的一部分。

无形参是一种书写代码的风格，这种风格移除了非必需的形参映射实参逻辑，其目的在于提高代码的可读性和可理解性。
