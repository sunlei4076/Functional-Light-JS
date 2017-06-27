# JavaScript轻量级函数式编程
# 第 4 章：组合函数

到目前为止，我希望你对理解在函数式编程中使用函数的重要性上，能感到舒服多了。

一个函数式编程者，会将他们程序中的每一个函数当成一小块简单的乐高部件。他们能一眼辨别出蓝色的2x2方块，并准确的知道它是如何工作的、能用它做些什么。当他们打算构建一个更大、更复杂的乐高模型时，当他们每一次需要下一块部件的时候，他们能够准确的做到从备用部件中找到并拿过来使用。

但有些时候，你把蓝色2x2的方块和灰色4x1的方块以某种形式组装到一起，然后你意识到：“这是个有用的部件，我可能会常用到它”。

那么你现在想到了一种新的“部件”，它是两种其他部件的组合，并且要求该部件在任何时候触手可及。这时候，将这个蓝-黑色L形状的方块组合体放到需要使用的地方，比每次分开考虑两种独立方块的组合要有效的多。

函数有多种多样的形状和大小。我们能够定义某种组合方式，来让它们成为一种新的组合函数，程序中不同的部分都可以使用这个函数。这种将函数一起使用的过程叫做组合。

## 输出 到 输入

我们已经见过几种组合的例子。比如，在第 3 章中，我们对 `unary(..)` 的讨论包含了如下表达式：`unary(adder(3))`。仔细想想这里发生了什么。

为了将两个函数整合起来，将第一个函数调用产生的输出当做第二个函数调用的输入。在 `unary(adder(3))` 中，`adder(3)` 的调用返回了一个值（值是一个函数）；该值被直接作为一个参数传入到 `unary(..)` 中，同样的，这个调用返回了一个值（值为另一个函数）。

让我们回放一下过程并且将数据流动的概念视觉化，是这个样子：

```
functionValue <-- unary <-- adder <-- 3
```

`3` 是 `adder(..)` 的输入。而 `adder(..)` 是 `unary(..)` 的输入。`unary(..)` 的输出是 `functionValue`。 它是 `unary(..)` 和 `adder(..)` 的组合。

把数据的流向想象成糖果工厂的一条传送带，每一次操作其实都是冷却、切割、包装糖果中的一步。在该章节中，我们将会用糖果工厂的类比来解释什么是组合。

<p align="center">
	<img src="fig2.png">
</p>

让我们一步一步的来了解组合。首先假设你程序中可能存在这么两个工具函数。

```js
function words(str) {
	return String( str )
		.toLowerCase()
		.split( /\s|\b/ )
		.filter( function alpha(v){
			return /^[\w]+$/.test( v );
		} );
}

function unique(list) {
	var uniqList = [];

	for (let i = 0; i < list.length; i++) {
		// value not yet in the new list?
		if (uniqList.indexOf( list[i] ) === -1 ) {
			uniqList.push( list[i] );
		}
	}

	return uniqList;
}
```

使用这两个函数来分析文本字符串：

```js
var text = "To compose two functions together, pass the \
output of the first function call as the input of the \
second function call.";

var wordsFound = words( text );
var wordsUsed = unique( wordsFound );

wordsUsed;
// ["to","compose","two","functions","together","pass",
// "the","output","of","first","function","call","as",
// "input","second"]
```

我们把 `words(..)` 输出的数组命名为 `wordsFound`。`unique(..)` 的输入也是一个数组，因此我们可以将 `wordsFound` 传入给它。

让我们重新回到糖果工厂的流水线：第一台机器接收的“输入”是融化的巧克力，它的“输出”是一堆成型且冷却的巧克力。流水线上的下一个机器将这堆巧克力作为它的“输入”，它的“输出”是一片片切好的巧克力糖果。下一步就是，流水线上的另一台机器将这些传送带上的小片巧克力糖果处理，并输出成包装好的糖果，准备打包和运输。

<img src="fig3.png" align="right" width="70" hspace="20">

糖果工厂靠这套流程运营的很成功，但是和所有的商业公司一样，管理者们需要不停的寻找增长点。

为了跟上更多糖果的生产需求，他们决定拿掉传送带这么个玩意，直接把三台机器叠在一起，这样第一台的输出阀就直接和下一台的输入阀直接连一起了。这样第一台机器和第二台机器之间，就再也不会有一堆巧克力在传送带上慢吞吞的移动了，并且也不会有空间浪费和隆隆的噪音声了。

这项革新为工厂节省了很大的空间，所以管理者很高兴，他们每天能够造更多的糖果了！

等价于升级后的糖果工厂配置的代码跳过了中间步骤（上面代码片段中的 `wordsFound` 变量），仅仅是将两个函数调用一起使用：

```js
var wordsUsed = unique( words( text ) );
```

**注意：** 尽管我们通常以从左往右的方式阅读函数调用 -- 先 `unique(..)` 然后 `words(..)` -- 这里的操作顺序实际上是从右往左的，或者说是自内而外。`words(..)` 将会首先运行，然后才是 `unique(..)`。晚点我们会讨论符合我们自然的、从左往右阅读执行顺序的模式，叫做 `pipe(..)`。

堆在一起的机器工作的还不错，但有一些太笨重了，电线挂的到处都是。创造的机器堆越多，工厂车间就会变得越凌乱。而且，装配和维护这些机器堆太占用时间了。

<img src="fig4.png" align="left" width="130" hspace="20">

有一天早上，一个糖果工厂的工程师突然想到了一个好点子。她想，如果她能在外面做一个大盒子把所有的电线都藏起来，效果肯定超级棒；盒子里面，三台机器相互连接，而盒子外面，一切都变得很整洁、干净。在这个很赞的机器的顶部，是倾倒融化巧克力的管道，在它的底部，是吐出包装好的巧克力糖果的管道。

这样一个单个的组合版机器，变得更易移动和安装到工厂需要的地方中去了。工厂的车间工人也会变得更高兴，因为他们不用再摆弄三台机子上的那些按钮和表盘了；他们很快更倾向于使用这个独立的很赞的机器。

回到代码上：我们现在了解到 `words(..)` 和 `unique(..)` 执行的特定顺序 -- 思考：对乐高的组合 -- 是一种我们在应用中其它部分也能够用到的东西。所以，现在让我们定义一个组合这些玩意的函数：

```js
function uniqueWords(str) {
	return unique( words( str ) );
}
```

`uniqueWords(..)` 接收一个字符串并返回一个数组。它是 `unique(..)` 和 `words(..)` 的组合，并且满足我们的数据流向要求：

```
wordsUsed <-- unique <-- words <-- text
```

你现在应该能够明白了：糖果工厂设计模式的演变革命就是函数的组合。

### 制造机器

糖果工厂一切运转良好，多亏了省下的空间，他们现在有足够多的地方来尝试制作新的糖果了。鉴于之前的成功，管理者迫切的想要发明新的棒棒的组合版机器，从而能打造更多的糖果种类。

但工厂的工程师们跟不上老板的节奏，因为每次造一台新的棒棒的组合版机器，他们就要花费很多的时间来造新的外壳，从而适应那些独立的机器。

所以工程师们联系了一家工业机器制供应商来帮他们。他们很惊讶的发现这家供应商竟然提供 **机器制造** 器！听起来好像不可思议，他们买入了这么一台机器。这台机器能够将工厂中小一点的机器 -- 负责巧克力冷却的、切割的，打个比方 -- 并把它们自动连线，甚至在外面还自动包了一层干净的盒子。这么牛的机器简直能把这家糖果工厂送上天了！

<p align="center">
	<img src="fig5.png" width="300">
</p>

回到代码上，让我们定义一个工具函数叫做 `compose2(..)`，它能够自动创建两个函数的组合，和我们手动做的是一模一样。

```js
function compose2(fn2,fn1) {
	return function composed(origValue){
		return fn2( fn1( origValue ) );
	};
}

// ES6箭头函数形式写法
var compose2 =
	(fn2,fn1) =>
		origValue =>
			fn2( fn1( origValue ) );
```

你是否注意到我们定义参数的顺序是 `fn2,fn1`，不仅如此，参数中列出的第二个函数（也被称作 `fn1`）会首先运行，然后参数中的第一个函数（`fn2`）？换句话说，这些函数是以从右往左的顺序组合的。

这看起来是种奇怪的实现，但这是有原因的。大部分传统的 FP 库为了顺序而将它们的 `compose(..)` 定义为从右往左的工作，所以我们沿袭了这种惯例。

但是为什么这么做？我认为最简单的解释（但不一定符合真实的历史）就是我们在以手动执行的书写顺序来列出它们，或是与我们从左往右阅读这个列表时看到它们的顺序相符合。

`unique(words(str))` 以从左往右的顺序列出了 `unique, words` 函数，所以我们让 `compose2(..)` 工具也以这种顺序接收它们。现在，更高效的糖果制造机定义如下：

```js
var uniqueWords = compose2( unique, words );
```

### 组合的变体

看起来貌似 `<-- unique <-- words` 的组合方式是这两种函数能够被组合起来的唯一顺序。但我们实际上能够以另外的目的创建一个工具，将它们以相反的顺序组合起来。

```js
var letters = compose2( words, unique );

var chars = letters( "How are you Henry?" );
chars;
// ["h","o","w","a","r","e","y","u","n"]
```

因为 `words(..)` 工具函数，上面的代码才能正常工作。为了值类型的安全，第一步使用 `String(..)` 将它的输入强转为一个字符串。所以 `unique(..)` 返回的数组 -- 现在是 `words(..)` 的输入 -- 成为了 `"H,o,w, ,a,r,e,y,u,n,?"` 这样的字符串。然后 `words(..)` 中的行为将字符串处理成为 `chars` 数组。

不得不承认，这是个刻意的例子。但重点是，函数的组合不总是无方向的。有时候我们将灰方块放到蓝方块上，有时我们又会将蓝方块放到最上面。

糖果工厂最好要小心点了，假如他们尝试将包装好的糖果放入搅拌和冷却巧克力的机器。

### 通用组合

如果我们能够定义两个函数的组合，我们也同样能够支持组合任意数量的函数。任意数目函数的组合的通用可视化数据流如下：

```
finalValue <-- func1 <-- func2 <-- ... <-- funcN <-- origValue
```

<p align="center">
	<img src="fig6.png" width="300">
</p>

现在糖果工厂拥有了最好的制造机：它能够接收任意数量独立的小机器，并吐出一个大只的、超赞的机器，能把每一步都按照顺序做好。这个糖果制作流程简直棒呆了！简直是威利·旺卡（译者注：《查理和巧克力工厂》中的人物，他拥有一座巧克力工厂）的梦想！

我们能够像这样实现一个通用 `compose(..)` 工具函数：

```js
function compose(...fns) {
	return function composed(result){
		// 拷贝一份保存函数的数组
		var list = fns.slice();

		while (list.length > 0) {
			// 将最后一个函数从列表尾部拿出
			// 并执行它
			result = list.pop()( result );
		}

		return result;
	};
}

// ES6箭头函数形式写法
var compose =
	(...fns) =>
		result => {
			var list = fns.slice();

			while (list.length > 0) {
				// 将最后一个函数从列表尾部拿出
				// 并执行它
				result = list.pop()( result );
			}

			return result;
		};
```

现在看一下组合超过两个函数的例子。回想下我们的 `uniqueWords(..)` 组合例子，让我们增加一个 `skipShortWords(..)`。

```js
function skipShortWords(list) {
	var filteredList = [];

	for (let i = 0; i < list.length; i++) {
		if (list[i].length > 4) {
			filteredList.push( list[i] );
		}
	}

	return filteredList;
}
```

让我们再定义一个 `biggerWords(..)` 来包含 `skipShortWords(..)`。我们期望等价的手工组合方式是 `skipShortWords(unique(words(text)))`，所以让我们照此来做一个 `compose(..)`：

```js
var text = "To compose two functions together, pass the \
output of the first function call as the input of the \
second function call.";

var biggerWords = compose( skipShortWords, unique, words );

var wordsUsed = biggerWords( text );

wordsUsed;
// ["compose","functions","together","output","first",
// "function","input","second"]
```

现在，让我们回忆一下第 3 章中出现的 `partialRight(..)` 来让组合变的更有趣。我们能够构造一个由 `compose(..)` 自身组成的右偏函数应用，通过提前定义好第二和第三参数（`unique(..)` 和 `words(..)`）；我们把它称作 `filterWords(..)`（如下）。

然后，我们能够通过多次调用 `filterWords(..)` 来完成组合，但是每次的第一参数却各不相同。

```js
// 注意： 使用 a `<= 4` 来检查，而不是 `skipShortWords(..)` 中用到的 `> 4`
function skipLongWords(list) { /* .. */ }

var filterWords = partialRight( compose, unique, words );

var biggerWords = filterWords( skipShortWords );
var shorterWords = filterWords( skipLongWords );

biggerWords( text );
// ["compose","functions","together","output","first",
// "function","input","second"]

shorterWords( text );
// ["to","two","pass","the","of","call","as"]
```

花些时间考虑一下基于 `compose(..)` 的右偏函数应用给了我们什么。它允许我们在组合的第一步之前做指定，然后以不同后期步骤的组合来创建特定的变体。这是函数式编程中最强大的手段之一。

你也能通过 `curry(..)` 作出的组合来替代偏函数应用，但因为从右往左的顺序，比起只使用 `curry( compose, ..)` 本身，你可能更想使用 `curry( reverseArgs(compose), ..)`。

**注意：** 因为 `curry(..)` 依赖于探测数目（`length`）或手动指定数目，而 `compose(..)` 是一个可变的函数，所以你需要手动指定数目，就像这样 `curry(.. , 3)`。

### 不同的实现

当然，你可能永远不会在生产中使用自己写的 `compose(..)`，而更倾向于使用某个库所提供的方案。但我发现了解底层工作的原理实际上对强化理解函数式编程中通用概念非常有用。

所以让我们看看对于 `compose(..)` 的不同实现方案。我们能看到每一种实现中有一些正反观点，特别是性能。

我们讲在文中稍后查看 `reduce(..)` 工具的细节，但现在，只需了解它将一个列表（数组）简化为一个单一的有限值。看起来像是一个很棒的循环体。

举个栗子，如果在数字列表 `[1,2,3,4,5,6]` 上做加法约减，你将要循环它们，并且随着循环将它们加在一起。这一过程将首先将 `1` 加 `2`，然后将结果加 `3`，然后加 `4`，等等。最后得到总和：`21`。

原始版本的 `compose(..)` 使用一个循环并且饥渴的（也就是，立刻）执行计算，从一个调用的结果传递到下一个调用。我们可以通过 `reduce(..)` （代替循环）做到同样的事。

```js
function compose(...fns) {
	return function composed(result){
		return fns.reverse().reduce( function reducer(result,fn){
			return fn( result );
		}, result );
	};
}

// ES6箭头函数形式写法
var compose = (...fns) =>
	result =>
		fns.reverse().reduce(
			(result,fn) =>
				fn( result )
			, result
		);
```

注意到 `reduce(..)` 循环发生在最后的 `composed(..)` 运行时，并且每一个中间的 `result(..)` 将会在下一次调用时作为输入值传递给下一个迭代。

这种实现的优点就是代码更简练，并且使用了常见的函数式编程结构：`reduce(..)`。这种实现方式的性能和原始的 `for` 循环版本很相近。 

但是，这种实现局限处在于外层的组合函数（也就是，组合中的第一个函数）只能接收一个参数。其他大多数实现在首次调用的时候就把所有参数传进去了。如果组合中的每一个函数都是一元的，这个方案没啥大问题。但如果你需要给第一个调用传递多参数，那么你可能需要不同的实现方案。

为了修正第一次调用的单参数限制，我们可以仍使用 `reduce(..)` ，但加一个懒执行函数包裹器：

```js
function compose(...fns) {
	return fns.reverse().reduce( function reducer(fn1,fn2){
		return function composed(...args){
			return fn2( fn1( ...args ) );
		};
	} );
}

// ES6箭头函数形式写法
var compose =
	(...fns) =>
		fns.reverse().reduce( (fn1,fn2) =>
			(...args) =>
				fn2( fn1( ...args ) )
		);
```

注意到我们直接返回了 `reduce(..)` 调用的结果，该结果自身就是个函数，不是一个计算过的值。**该**函数让我们能够传入任意数目的参数，在整个组合过程中，将这些参数传入到第一个函数调用中，然后依次产出结果给到后面的调用。

相较于直接计算结果并把它传入到 `reduce(..)` 循环中进行处理，这种实现通过在组合之前只运行 **一次** `reduce(..)` 循环，然后将所有的函数调用运算全部延迟了 -- 成为懒运算。每一个简化后的局部结果都是一个包裹层级更多的函数。

当你调用最终组合函数并且提供一个或多个函数的时候，这个层层嵌套的大函数内部的所有层级，由内而外调用，以相反的方式连续执行（不是通过循环）。

这个版本的性能特征和之前 `reduce(..)` 基础实现版有潜在的差异。在这儿，`reduce(..)` 只在生成大个的组合函数时运行过一次，然后这个组合函数只是简单的一层层执行它内部所嵌套的函数。在前一版本中，`reduce(..)` 将在每一次调用中运行。

在考虑哪一种实现更好时，你的情况可能会不一样，但是要记得后面的实现方式并没有像前一种限制只能传一个参数。

我们也能够使用递归来定义 `compose(..)`。递归式定义 `compose(fn1,fn2, .. fnN)` 看起来会是这样：

```
compose( compose(fn1,fn2, .. fnN-1), fnN );
```

**注意：** 我们将在第 9 章揭示更多的细节，所以如果这块看起来让你疑惑，那么暂时跳过该部分是没问题的，可以在阅读完第 9 章后再来看。

这里是我们如何用递归实现 `compose(..)` 的代码：

```js
function compose(...fns) {
	// 拿出最后两个参数
	var [ fn1, fn2, ...rest ] = fns.reverse();

	var composedFn = function composed(...args){
		return fn2( fn1( ...args ) );
	};

	if (rest.length == 0) return composedFn;

	return compose( ...rest.reverse(), composedFn );
}

// ES6箭头函数形式写法
var compose =
	(...fns) => {
		// 拿出最后两个参数
		var [ fn1, fn2, ...rest ] = fns.reverse();

		var composedFn =
			(...args) =>
				fn2( fn1( ...args ) );

		if (rest.length == 0) return composedFn;

		return compose( ...rest.reverse(), composedFn );
	};
```

我认为递归实现的好处是更加概念化。我个人觉得相较于不得不在循环里跟踪运行结果，通过递归的方式进行重复的动作反而更容易想清楚。所以我更喜欢以这种方式的代码来表达。

Others will find the recursive approach quite a bit more daunting to mentally juggle. I invite you to make your own evaluations.
其他人可能会觉得递归的方法在智力上造成的困扰更让人有些畏惧。我建议你作出自己的评估。

## 重排序组合

我们早期谈及的是从右往左顺序的标准 `compose(..)` 实现。这么做的好处是能够和手工组合列出参数（函数）的顺序保持一致。

不足就是它们列出的顺序和它们执行的顺序是相反的，这将会造成困扰。同时，不得不使用 `partialRight(compose, ..)` 提早定义要在组合过程中 **第一个** 执行的函数。

相反的顺序，从右往左的组合，有个常见的名字：`pipe(..)`。这个名字据说来自 Unix/Linux 界，那里大量的程序通过“管道传输”（`|` 运算符）第一个的输出到第二个的输入，等等（即，`ls -la | grep "foo" | less`）。

`pipe(..)` 与 `compose(..)` 一模一样，除了它将列表中的函数从左往右处理。

```js
function pipe(...fns) {
	return function piped(result){
		var list = fns.slice();

		while (list.length > 0) {
			// take the first function from the list
			// and execute it
			result = list.shift()( result );
		}

		return result;
	};
}
```

实际上，我们只需将 `compose(..)` 的参数反转就能定义出来一个 `pipe(..)`。

```js
var pipe = reverseArgs( compose );
```

非常简单！

回忆下之前的通用组合的例子：

```js
var biggerWords = compose( skipShortWords, unique, words );
```

以 `pipe(..)` 的方式来实现，我们只需要反转参数的顺序：

```js
var biggerWords = pipe( words, unique, skipShortWords );
```

 `pipe(..)` 的优势在于它以函数执行的顺序排布参数，某些情况下能够减轻阅读者的疑惑。`pipe(words,unique,skipShortWords)` 看起来和读起来会更简单，能知道我们首先执行 `words(..)`，然后 `unique(..)`，最后是 `skipShortWords(..)`。

假如你想要应用**第一个**函数（们）来负责执行，`pipe(..)` 同样也很方便。就像我们之前使用 `compose(..)` 构建的右偏函数应用一样。

对比：

```js
var filterWords = partialRight( compose, unique, words );

// vs

var filterWords = partial( pipe, words, unique );
```

你可能会回想起第 3 章 `partialRight(..)` 中的定义，它实际使用了 `reverseArgs(..)`，就像我们的 `pipe(..)` 现在所做的。所以，不管怎样，我们得到了同样的结果。

在这一特定场景下使用 `pipe(..)` 的轻微性能优势在于我们不必再通过右偏函数应用的方式来使用 `compose(..)` 保存从右往左的参数顺序，使用
 `pipe(..)` 我们不必再去将参数顺序反转回去，就像我们在 `partialRight(..)` 中所做的一样。所以在这里 `partial(pipe, ..)` 比 `partialRight(compose, ..)` 要好一点。

一般来说，在使用一个完善的函数式编程库时，`pipe(..)` 和 `compose(..)` 没有明显的性能区别。

## 抽象

抽象经常被定义为对两个或多个任务公共部分的剥离。通用部分只定义一次，从而避免重复。为了展现每个任务的特殊部分，通用部分需要被参数化。

举个例子，思考如下（明显刻意生成的）代码：

```js
function saveComment(txt) {
	if (txt != "") {
		comments[comments.length] = txt;
	}
}

function trackEvent(evt) {
	if (evt.name !== undefined) {
		events[evt.name] = evt;
	}
}
```

这两个函数都是将一个值存入一个数据源，这是通用的部分。不同的是一个是将值放置到数组的末尾，另一个是将值放置到对象的某个属性上。

让我们抽象一下：

```js
function storeData(store,location,value) {
	store[location] = value;
}

function saveComment(txt) {
	if (txt != "") {
		storeData( comments, comments.length, txt );
	}
}

function trackEvent(evt) {
	if (evt.name !== undefined) {
		storeData( events, evt.name, evt );
	}
}
```

引用一个对象（或数组，多亏了 JS 中方便的 `[]` 符号）属性和将值设入的通用任务被抽象到独立的 `storeData(..)` 函数。这个函数当前只有一行，该函数能提出其它多任务中通用的行为，比如生成唯一的数字 ID 或将时间戳存入。

如果我们在多处重复通用的行为，我们将会面临改了几处但忘了改别处的维护风险。在做这类抽象时，有一个原则是，通常被称作 DRY（don't repeat yourself）。

DRY 力求能在程序的任何任务中有唯一的定义。代码不够 DRY 的另一个托辞就是程序员们太懒，不想做非必要的工作。 

抽象能够走得更远。思考：

```js
function conditionallyStoreData(store,location,value,checkFn) {
	if (checkFn( value, store, location )) {
		store[location] = value;
	}
}

function notEmpty(val) { return val != ""; }

function isUndefined(val) { return val === undefined; }

function isPropUndefined(val,obj,prop) {
	return isUndefined( obj[prop] );
}

function saveComment(txt) {
	conditionallyStoreData( comments, comments.length, txt, notEmpty );
}

function trackEvent(evt) {
	conditionallyStoreData( events, evt.name, evt, isPropUndefined );
}
```

为了实现 DRY 和避免重复的 `if` 语句，我们将条件判断移动到了通用抽象中。我们同样假设在程序中其它地方可能会检查非空字符串或非 `undefined` 的值，所以我们也能将这些东西 DRY 出来。

这些代码现在**变得**更 DRY 了，但有些过度扩展了。开发者需要对他们程序中每个部分使用恰当的抽象级别保持谨慎，不能太过，也不能不够。

关于我们在该章中对函数的组合进行的大量讨论，看起来它的好处是实现这种 DRY 抽象。但让我们别急着下结论，因为我认为组合实际上在我们的代码中发挥着更重要的作用。

而且，**即使某些东西只出现了一次，组合仍然十分有用** （没有可以被抽出来的东西）。

除了通用化和特殊化的对比，我认为抽象有更多有用的定义，正如下面这段引用所说：

> ... 抽象是一个过程，程序员将一个名字与潜在的复杂程序片段关联起来，这样该名字就能够被认为代表函数的目的，而不是代表函数如何实现的。通过隐藏无关的细节，抽象降低了概念复杂度，让程序员在任意时间都可以集中注意力在程序内容中的可维护子集上。
>
> 程序设计语言， 迈克尔 L 斯科特
>
> https://books.google.com/books?id=jM-cBAAAQBAJ&pg=PA115&lpg=PA115&dq=%22making+it+possible+for+the+programmer+to+focus+on+a+manageable+subset%22&source=bl&ots=yrJ3a-Tvi6&sig=XZwYoWwbQxP2w5qh2k2uMAPj47k&hl=en&sa=X&ved=0ahUKEwjKr-Ty35DSAhUJ4mMKHbPrAUUQ6AEIIzAA#v=onepage&q=%22making%20it%20possible%20for%20the%20programmer%20to%20focus%20on%20a%20manageable%20subset%22&f=false

// TODO: 给这本书或引用弄一个更好的参照，至少找到一个更好的在线链接

这段引用表述的观点是抽象 -- 通常来说，是指把一些代码片段放到自己的函数中 -- 是围绕着能将两部分功能分离，从而达到可以专注于某一独立的部分为主要目的来服务的。

需要注意的是，这种场景下的抽象并不是为了**隐藏**细节，比如把一些东西当作黑盒来对待。这一观念其实更贴近于编程中的封装性原则。**我们不是通过抽象来隐藏细节，而是通过分离来突出关注点**。

还记得这段文章的开头，我说函数式编程的目的是为了创造更可读、更易理解的代码。一个有效的方法是将交织缠绕 -- 紧紧编织在一起，像一股绳子 -- 的代码解绑为分离的、更简单的 -- 松散绑定的 -- 代码片段。以这种方式来做的话，代码的阅读者将不会在寻找其它部分细节的时候被其中某块的细节所分心。

我们更高的目标是只对某些东西实现一次，这是 DRY 的观念。实际上，有些时候我们确实在代码中不断重复。于是，我们寻求更分离的实现方式。我们尝试突出关注点，因为这能提高可读性。

用另一种方式描述这个目标就是命令式 vs 声明式编程风格的区别。命令式代码主要关心的是描述**怎么做**来准确完成一项任务。声明式代码则是描述输出应该**是什么**，并将具体实现交给其它部分。

换句话说，声明式代码从**怎么做**中抽象出了**是什么**。尽管普通的声明式代码在可读性上强于命令式，但没有程序（除了机器码 1 和 0）是完全的声明式或者命令式代码。编程者必须在它们之间寻找平衡。

ES6 增加了很多语法功能，能将老的命令式操作转换为新的声明式形式。可能最清晰的当属解构了。解构是一种赋值模式，它描述了如何将组合值（对象、数组）内的构成值分解出来的方法。

这里是一个数组结构的例子：

```js
function getData() {
	return [1,2,3,4,5];
}

// 命令式
var tmp = getData();
var a = tmp[0];
var b = tmp[3];

// 声明式
var [ a ,,, b ] = getData();
```

**是什么**就是将数组中的第一个值赋给 `a`，然后第四个值赋给 `b`。**怎么做**就是得到一个数组的引用（`tmp`）然后手动的通过数组索引 `0` 和 `3`，分别赋值给 `a` 和 `b`。

数组的解构是否**隐藏**了赋值细节？这要看你看待的角度了。我认为它知识简单的将**是什么**从**怎么做**中分离出来。JS 引擎仍然做了赋值的工作，但它阻止了你自己去抽象**怎么做**的过程。

相反的是，你阅读 `[ a ,,, b ] = ..` 的时候，便能看到该赋值模式只不过是告诉你将要发生的**是什么**。数组的解构是声明式抽象的一个例子。

### 将组合当作抽象

函数组合到底做了什么？函数组合同样也是一种声明式抽象。

回想下之前的 `shorterWords(..)` 例子。让我们对比下命令式和声明式的定义。

```js
// 命令式
function shorterWords(text) {
	return skipLongWords( unique( words( text ) ) );
}

// 声明式
var shorterWords = compose( skipLongWords, unique, words );
```

声明式关注点在**是什么**上 -- 这 3 个函数传递的数据从一个字符串到一系列更短的单词 -- 并且将**怎么做**留在了 `compose(..)`的内部。

在一个更大的层面上看，`shorterWords = compose(..)` 行解释了**怎么做**来定义一个 `shorterWords(..)` 工具函数，这样在代码的别处使用时，只需关注下面这行声明式的代码输出**是什么**。

```js
shorterWords( text );
```

组合将一步步得到一系列更短的单词的过程抽象了出来。

相反的看，如果我们不使用组合抽象呢？

```js
var wordsFound = words( text );
var uniqueWordsFound = unique( wordsFound );
skipLongWords( uniqueWordsFound );
```

或者这种：

```js
skipLongWords( unique( words( text ) ) );
```

这两个版本展示的都是一种更加命令式的风格，违背了声明式风格优先原则。阅读者关注这两个代码片段时，会被更多的要求了解**怎么做**而不是**是什么**。

函数组合并不是通过 DRY 的原则来节省代码量。即使 `shorterWords(..)` 的使用只出现了一次 -- 所以并没有重复问题需要避免！-- 从**怎么做**中分离出**是什么**仍能帮助我们提升代码。

组合是一个抽象的强力工具，它能够将命令式代码抽象为更可读的声明式代码。

## 回顾“点”

既然我们已经把组合都了解了一遍 -- 那么是时候抛出函数式编程中很多地方都有用的小技巧了 -- 让我们通过在某个场景下回顾第 3 章的“无点”（译者注：“无点”指的是移除对函数形参的引用）段落中的无点代码，并把它重构的稍微复杂点来观察这种小技巧。

```js
// 提供该API：ajax( url, data, cb )
var getPerson = partial( ajax, "http://some.api/person" );
var getLastOrder = partial( ajax, "http://some.api/order", { id: -1 } );

getLastOrder( function orderFound(order){
	getPerson( { id: order.personId }, function personFound(person){
		output( person.name );
	} );
} );
```

我们想要移除的“点”是对 `order` 和 `person` 参数的引用。

让我们尝试将 `person` “点”移出 `personFound(..)` 函数。要达到目的，我们需要首先定义：

```js
function extractName(person) {
	return person.name;
}
```

但据我们观察这段操作能够表达的更通用些：将任意对象的任意属性通过属性名提取出来。让我们把这个工具称为 `prop(..)`：

```js
function prop(name,obj) {
	return obj[name];
}

// ES6 箭头函数形式
var prop =
	(name,obj) =>
		obj[name];
```

我们处理对象属性的时候，也需要定义下反操作的工具：`setProp(..)`，为了将属性值设入某个对象内。

但是，我们想小心一些，不改动现存的对象，而是创建一个携带变化的复制对象，并将它返回出去。关于这块的处理将在第 5 章中讨论更多细节。

```js
function setProp(name,obj,val) {
	var o = Object.assign( {}, obj );
	o[name] = val;
	return o;
}
```

现在，定义一个 `extractName(..)` ，它能将对象中的 `"name"` 属性拿出来，我们将部分应用 `prop(..)`：

```
var extractName = partial( prop, "name" );
```

**注意:** 不要误解这里的 `extractName(..)`，它其实什么都还没有做。我们只是部分应用 `prop(..)` 来创建了一个等待接收包含 `"name"`属性的对象的函数。我们也能通过`curry(prop)("name")`做到一样的事。

下一步，让我们缩小关注点，看下例子中嵌套的这块查找操作的调用：

```js
getLastOrder( function orderFound(order){
	getPerson( { id: order.personId }, outputPersonName );
} );
```

我们该如何定义 `outputPersonName(..)`？为了方便形象化我们所需要的东西，想一下我们需要的数据流是什么样：

```
output <-- extractName <-- person
```

`outputPersonName(..)` 需要是一个接收（对象）值的函数，并将它传递给 `extractName(..)`，然后将处理后的值传给 `output(..)`。

希望你能看出这里需要 `compose(..)` 操作。所以我们能够将 `outputPersonName(..)` 定义为：

```
var outputPersonName = compose( output, extractName );
```

我们刚刚创建的 `outputPersonName(..)` 函数是提供给 `getPerson(..)` 的回调。所以我们还能定义一个函数叫做 `processPerson(..)` 来处理回调参数，使用 `partialRight(..)`：

```js
var processPerson = partialRight( getPerson, outputPersonName );
```

让我们用新函数来重构下之前的代码：

```js
getLastOrder( function orderFound(order){
	processPerson( { id: order.personId } );
} );
```

唔，进展还不错！

但我们需要继续移除掉 `order` 这个“点”。下一步是观察 `personId` 能够被 `prop(..)` 从一个对象（比如 `order`）中提取出来，就想我们在 `person` 对象中提取 `name` 一样。

```js
var extractPersonId = partial( prop, "personId" );
```

为了创建传递给 `processPerson(..)` 的对象（ `{ id: .. }` 的形式），让我们创建一个工具 `makeObjProp(..)`，用来以特定的属性名将值包装为一个对象。

```js
function makeObjProp(name,value) {
	return setProp( name, {}, value );
}

// ES6 箭头函数形式
var makeObjProp =
	(name,value) =>
		setProp( name, {}, value );
```

**提示：** 这个工具在 Ramda 库中被成为 `objOf(..)`。

就想我们之前使用 `prop(..)` 来创建 `extractName(..)`，我们将部分应用 `makeObjProp(..)` 来创建 `personData(..)` 函数用来制作我们的数据对象。

```js
var personData = partial( makeObjProp, "id" );
```

为了使用 `processPerson(..)` 来完成通过 `order` 值查找一个人的功能，我们需要的数据流如下：

```
processPerson <-- personData <-- extractPersonId <-- order
```

所以我们只需要在使用一次 `compose(..)` 来定义一个 `lookupPerson(..)` ：

```js
var lookupPerson = compose( processPerson, personData, extractPersonId );
```

然后，就是这样了！把这整个例子重新组合起来，不带任何的“点”：

```js
var getPerson = partial( ajax, "http://some.api/person" );
var getLastOrder = partial( ajax, "http://some.api/order", { id: -1 } );

var extractName = partial( prop, "name" );
var outputPersonName = compose( output, extractName );
var processPerson = partialRight( getPerson, outputPersonName );
var personData = partial( makeObjProp, "id" );
var extractPersonId = partial( prop, "personId" );
var lookupPerson = compose( processPerson, personData, extractPersonId );

getLastOrder( lookupPerson );
```

哇哦。没有点。并且 `compose(..)` 在两处地方看起来相当有用！

我认为在这样的场景下，即使推导出我们最终答案的步骤有些多，但最终的代码却变得更加可读，因为我们不用再去详细的调用每一步了。

假如你不想看到/命名这么多中间步骤，你已然可以通过不使用独立变量而是将表达式串起来来来保留无点特性。

```js
partial( ajax, "http://some.api/order", { id: -1 } )
(
	compose(
		partialRight(
			partial( ajax, "http://some.api/person" ),
			compose( output, partial( prop, "name" ) )
		),
		partial( makeObjProp, "id" ),
		partial( prop, "personId" )
	)
);
```

这段代码肯定没那么罗嗦了，但我认为比之前的每个操作都有其对应的变量相比，可读性略有降低。但是不管怎样，组合帮助我们实现了无点的风格。

## 总结

函数组合是一种定义函数的模式，它能将一个函数调用的输出路由到另一个函数的调用上，然后一直进行下去。

因为 JS 函数只能返回单个值，这个模式本质上要求所有组合中的函数（可能第一个调用的函数除外）是一元的，从上一个输出中只接收一个输入。

相较于在我们的代码里详细列出每个调用，函数组合使用 `compose(..)` 工具来提取出实现细节，让代码变得更可读，让我们更关注组合完成的**是什么**，而不是它具体**做什么**。

组合 -- 声明式数据流 -- 是支撑函数式编程其他特性的最重要的工具之一。
