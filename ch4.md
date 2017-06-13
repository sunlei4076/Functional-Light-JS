# Functional-Light JavaScript
# Chapter 4: Composing Functions
# 第 4 章：组合函数

By now, I hope you're feeling much more comfortable with what it means to use functions for functional programming.
到目前为止，我希望你对在函数式编程中使用函数的重要性上，能感到更舒服些。

A functional programmer sees every function in their program like a little simple lego piece. They recognize the blue 2x2 brick at a glance, and know exactly how it works and what they can do with it. As they go about building a bigger complex lego model, as they need each next piece, they already have an instinct for which of their many spare pieces to grab.
一个函数式编程者，会将他们程序中的每一个函数当成一小块简单的乐高部件。他们能一眼辨别出蓝色的2x2方块，并准确的知道它是如何工作的、能用它做些什么。当他们打算构建一个更大、更复杂的乐高模型时，当他们每一次需要下一块部件的时候，他们能够准确的做到从备用部件中找到并拿过来使用。

But sometimes you take the blue 2x2 brick and the gray 4x1 brick and put them together in a certain way, and you realize, "that's a useful piece that I need often".
但有些时候，你把蓝色2x2的方块和灰色4x1的方块以某种形式组装到一起，然后你意识到：“这是个有用的部件，我可能会常用到它”。

So now you've come up with a new "piece", a combination of two other pieces, and you can reach for that kind of piece now anytime you need it. It's more effective to recognize and use this compound blue-gray L-brick thing where it's needed than to separately think about assembling the two individual bricks each time.
那么你现在想到了一种新的“部件”，它是两种其他部件的组合，并且要求该部件在任意时间触手可及。这时候，将这个蓝-黑色L形状的方块组合体放到需要使用的地方，比每次分开考虑两种独立方块的组合要有效的多。

Functions come in a variety of shapes and sizes. And we can define a certain combination of them to make a new compound function that will be handy in various parts of the program. This process of using functions together is called composition.
函数有多种多样的形状和大小。我们能够以定义某种组合方式，来让它们成为一种新的组合函数，程序中不同的部分都可以使用这个函数。这种将函数一起使用的过程叫做组合。

## Output To Input
## 输出 到 输入

We've already seen a few examples of composition. For example, in Chapter 3 our discussion of `unary(..)` included this expression: `unary(adder(3))`. Think about what's happening there.
我们已经见过几种组合的例子。比如，在第 3 章中，我们对 `unary(..)` 的讨论，包含了如下表达式：`unary(adder(3))`。仔细想想这里发生了什么。

To compose two functions together, pass the output of the first function call as the input of the second function call. In `unary(adder(3))`, the `adder(3)` call returns a value (a function); that value is directly passed as an argument to `unary(..)`, which also returns a value (another function).
为了将两个函数整合起来，将第一个函数调用产生的输出当做第二个函数调用的输入。在 `unary(adder(3))` 中，`adder(3)` 的调用返回了一个值（值是一个函数）；该值被直接作为一个参数传入到 `unary(..)` 中，同样的，这个调用返回了一个值（值为另一个函数）。

To take a step back and visualize the conceptual flow of data, consider:
让我们回放一下过程并且将数据流动的概念视觉化，是这个样子：

```
functionValue <-- unary <-- adder <-- 3
```

`3` is the input to `adder(..)`. The output of `adder(..)` is the input to `unary(..)`. The output of `unary(..)` is `functionValue`. This is the composition of `unary(..)` and `adder(..)`.
`3` 是 `adder(..)` 的输入。而 `adder(..)` 是 `unary(..)` 的输入。`unary(..)` 的输出是 `functionValue`。 它是 `unary(..)` 和 `adder(..)` 的组合。

Think of this flow of data like a conveyor belt in a candy factory, where each operation is a step in the process of cooling, cutting, and wrapping a piece of candy. We'll use the candy factory metaphor throughout this chapter to explain what composition is.
把数据的流向想象成糖果工厂的一条传送带，每一次操作其实都是冷却、切割、包装糖果中的一步。在该章节中，我们将会用糖果工厂的类比来解释什么是组合。

<p align="center">
	<img src="fig2.png">
</p>

Let's examine composition in action one step at a time. Consider these two utilitites you might have in your program:
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

To use these two utilities to analyze a string of text:
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

We name the array output of `words(..)` as `wordsFound`. The input of `unique(..)` is also an array, so we can pass the `wordsFound` into it.
我们把 `words(..)` 输出的数组命名为 `wordsFound`。`unique(..)` 的输入也是一个数组，因此我们可以将 `wordsFound` 传入给它。

Back to the candy factory assembly line: the first machine takes as "input" the melted chocolate, and its "output" is a chunk of formed and cooled chocolate. The next machine a little down the assembly line takes as its "input" the chunk of chocolate, and its "output" is a cut-up of piece of chocolate candy. Next, a machine on the line takes small pieces of chocolate candy from the conveyor belt and outputs wrapped candies ready to bag and ship.
让我们重新回到糖果工厂的流水线：第一台机器接收的“输入”是融化的巧克力，它的“输出”是一堆成型且冷却的巧克力。流水线上的下一个机器将这堆巧克力作为它的“输入”，它的“输出”是一片片切好的巧克力糖果（译者注：作者一定很喜欢吃糖，而且是巧克力）。下一步就是，流水线上的另一台机器将这些传送带上的小片巧克力糖果处理，并输出成包装好的糖果，准备打包和运输。

<img src="fig3.png" align="right" width="70" hspace="20">

The candy factory is fairly successful with this process, but as with all businesses, management keeps searching for ways to grow.
糖果工厂靠这套流程运营的很成功，但是和所有的商业公司一样，管理者们需要不停的寻找增长点。

To keep up with demand for more candy production, they decide to take out the conveyor belt contraption and just stack all three machines on top of each other, so that the output valve of one is connected directly to the input valve of the one below it. There's no longer sprawling wasted space where a chunk of chocolate slowly and noisly rumbles down a conveyor belt from the first machine to the second.
为了跟上更多糖果的生产需求，他们决定拿掉传送带这么个玩意，直接把三台机器叠在一起，这样第一台的输出阀就直接和下一台的输入阀直接连一起了。这样第一台机器和第二台机器之间，就再也不会有一堆巧克力在传送带上慢吞吞而导致的空间浪费和隆隆的噪音声了。

This innovation saves a lot of room on the factory floor, so management is happy they'll get to make more candy each day!
这项革新为工厂节省了很大的空间，所以管理者很高兴，他们们天能够造更多的糖果了！

The code equivalent of this improved candy factory configuration is to skip the intermediate step (the `wordsFound` variable in the earlier snippet), and just use the two function calls together:
等价于升级后的糖果工厂配置的代码跳过了中间步骤（上面代码片段中的 `wordsFound` 变量），仅仅是将两个函数调用一起使用：

```js
var wordsUsed = unique( words( text ) );
```

**Note:** Though we typically read the function calls left-to-right -- `unique(..)` and then `words(..)` -- the order of operations will actually be more right-to-left, or inner-to-outer. `words(..)` will run first and then `unique(..)`. Later we'll talk about a pattern that matches the order of execution to our natural left-to-right reading, called `pipe(..)`.
**注意：** 尽管我们通常以从左往右的方式阅读函数调用 -- 先 `unique(..)` 然后 `words(..)` -- 这里的操作顺序实际上是从右往左的，或者说是自内而外。`words(..)` 将会首先运行，然后才是 `unique(..)`。晚点我们会讨论符合我们自然的、从左往右阅读执行顺序的模式，叫做 `pipe(..)`。

The stacked machines are working fine, but it's kind of clunky to have the wires hanging out all over the place. The more of these machine-stacks they create, the more cluttered the factory floor gets. And the effort to assemble and maintain all these machine stacks is awfully time intensive.
堆在一起的机器工作的还不错，但有一些太笨重了，电线挂的到处都是。他们创造的机器堆越多，工厂车间就会变得越凌乱。而且，装配和维护这些机器堆太占用时间了。

<img src="fig4.png" align="left" width="130" hspace="20">

One morning, an engineer at the candy factory has a great idea. She figures that it'd be much more efficient if she made an outer box to hide all the wires; on the inside, all three of the machines are hooked up together, and on the outside everything is now neat and tidy. On the top of this fancy new machine is a valve to pour in melted chocolate and on the bottom is a valve that spits out wrapped chocolate candies. Brilliant!
有一天早上，一个糖果工厂的工程师突然想到了一个好点子。她想，如果她能在外面做一个大盒子把所有的电线都藏起来，效果肯定超级棒；盒子里面，三台机器相互连接，而盒子外面，一切都变得很整洁、干净。在这个很赞的机器的顶部，是倾倒融化巧克力的管道，在它的底部，是吐出包装好的巧克力糖果的管道。

This single compound machine is much easier to move around and install wherever the factory needs it. The workers on the factory floor are even happier because they don't need to fidget with buttons and dials on three individual machines anymore; they quickly prefer using the single fancy machine.
这样一个单个的组合版机器，变得更易移动和安装到工厂需要的地方中去。工厂的车间工人也会变得更高兴，因为他们不用再摆弄三台机子上的那些按钮和表盘了；他们很快更倾向于使用这个独立的很赞的机器。

Relating back to the code: we now realize that the pairing of `words(..)` and `unique(..)` in that specific order of execution -- think: compound lego -- is something we could use in several other parts of our application. So, let's define a compound function that combines them:
回到代码上：我们现在了解到 `words(..)` 和 `unique(..)` 执行的特定顺序 -- 思考： 对乐高的组合 -- 是一种我们在应用中其它部分也能够用到的东西。所以，现在让我们定义一个组合这些玩意的函数：

```js
function uniqueWords(str) {
	return unique( words( str ) );
}
```

`uniqueWords(..)` takes a string and returns an array. It's a composition of `unique(..)` and `words(..)`, as it fulfills the data flow:
`uniqueWords(..)` 接收一个字符串并返回一个数组。它是 `unique(..)` 和 `words(..)` 的组合，并且满足我们的数据流向要求：

```
wordsUsed <-- unique <-- words <-- text
```

You certainly get it by now: the unfolding revolution in candy factory design is function composition.
你现在应该能够明白了： 糖果工厂设计模式的演变革命就是函数的组合。

### Machine Making
### 机器制造

The candy factory is humming along nicely, and thanks to all the saved space, they now have plenty of room to try out making new kinds of candies. Building on the earlier success, management is keen to keep inventing new fancy compound machines for their growing candy assortment.
糖果工厂一切运转良好，多亏了省下的空间，他们现在有足够多的地方来尝试制作新的糖果了。鉴于之前的成功，管理者迫切的想要发明新的棒棒的组合版机器，从而能打造更多的糖果种类。

But the factory engineers struggle to keep up, because each time a new kind of fancy compound machine needs to be made, they spend quite a bit of time making the new outer box and fitting the individual machines into it.
但工厂的工程师们跟不上老板的节奏，因为每次造一台新的棒棒的组合版机器，他们就要花费很多的时间来造新的外壳，从而适应那些独立的机器。

So the factory engineers contact an industrial machine vendor for help. They're amazed to find out that this vendor offers a **machine-making** machine! As incredible as it sounds, they purchase a machine that can take a couple of the factory's smaller machines -- the chocolate cooling one and the cutting one, for example -- and wire them together automatically, even wrapping a nice clean bigger box around them. This is surely going to make the candy factory really take off!
所以工程师们联系了一家工业机器制供应商来帮他们。他们很惊讶的发现这家供应商竟然提供 **机器制造** 器！听起来好像不可思议，他们买入了这么一台机器。这台机器能够将工厂中小一点的机器 -- 负责巧克力冷却的、切割的，打个比方 -- 并把它们自动连线，甚至在外面还自动包了一层干净的盒子。这么牛的机器简直能把这家糖果工厂送上天了！

<p align="center">
	<img src="fig5.png" width="300">
</p>

Back to code land, let's consider a utility called `compose2(..)` that creates a composition of two functions automatically, exactly the same way we did manually:
回到代码上，让我们定义一个工具函数叫做 `compose2(..)`，它能够自动创建两个函数的组合，和我们手动做的是一模一样。

```js
function compose2(fn2,fn1) {
	return function composed(origValue){
		return fn2( fn1( origValue ) );
	};
}

// or the ES6 => form
// ES6箭头函数形式写法
var compose2 =
	(fn2,fn1) =>
		origValue =>
			fn2( fn1( origValue ) );
```

Did you notice that we defined the parameter order as `fn2,fn1`, and furthermore that it's the second function listed (aka `fn1` parameter name) that runs first, then the first function listed (`fn2`)? In other words, the functions compose from right-to-left.
你是否注意到我们定义参数的顺序是 `fn2,fn1`，不仅如此，参数中列出的第二个函数（也被称作 `fn1`）会首先运行，然后参数中的第一个函数（`fn2`）？换句话说，这些函数是以从右往左的顺序组合的。

That may seem like a strange choice, but there are some reasons for it. Most typical FP libraries define their `compose(..)` to work right-to-left in terms of ordering, so we're sticking with that convention.
这看起来是种奇怪的实现，但这是有原因的。大部分传统的 FP 库为了顺序而将它们的 `compose(..)` 定义为从右往左的工作，所以我们沿袭了这种惯例。

But why? I think the easiest explanation (but perhaps not the most historically accurate) is that we're listing them to match the order they are written if done manually, or rather the order we encounter them when reading from left-to-right.
但是为什么这么做？我认为最简单的解释（但不一定符合真实的历史）就是我们在以手动执行的书写顺序来列出它们，或是与我们从左往右阅读这个列表时看到它们的顺序相符合。

`unique(words(str))` lists the functions in the left-to-right order `unique, words`, so we make our `compose2(..)` utility accept them in that order, too. Now, the more efficient definition of the candy making machine is:
`unique(words(str))` 以从左往右的顺序列出了 `unique, words` 函数，所以我们让 `compose2(..)` 工具也以这种顺序接收它们。现在，更高效的糖果制造机定义如下：

```js
var uniqueWords = compose2( unique, words );
```

### Composition Variation
### 组合的变体

It may seem like the `<-- unique <-- words` combination is the only order these two functions can be composed. But we could actually compose them in the opposite order to create a utility with a bit of a different purpose:
看起来貌似 `<-- unique <-- words` 的组合方式是这两种函数能够被组合起来的唯一顺序。但我们实际上能够以另外的目的创建一个工具，将它们以相反的顺序组合起来。

```js
var letters = compose2( words, unique );

var chars = letters( "How are you Henry?" );
chars;
// ["h","o","w","a","r","e","y","u","n"]
```

This works because the `words(..)` utility, for value-type safety sake, first coerces its input to a string using `String(..)`. So the array that `unique(..)` returns -- now the input to `words(..)` -- becomes the string `"H,o,w, ,a,r,e,y,u,n,?"`, and then the rest of the behavior in `words(..)` processes that string into the `chars` array.
因为 `words(..)` 工具函数，上面的码才能正常工作。为了值类型的安全，第一步使用 `String(..)`将它的输入强转为一个字符串。所以 `unique(..)` 返回的数组 -- 现在是 `words(..)` 的输入 -- 成为了 `"H,o,w, ,a,r,e,y,u,n,?"` 这样的字符串。然后 `words(..)` 中的行为将字符串处理成为 `chars` 数组。

Admittedly, this is a contrived example. But the point is that function compositions are not always unidirectional. Sometimes we put the gray brick on top of the blue brick, and sometimes we put the blue brick on top.
不得不承认，这是个刻意的例子。但重点是，函数的组合不总是无方向的。有时候我们将灰方块放到蓝方块上，有时我们又会将蓝方块放到最上面。

The candy factory better be careful if they try to feed the wrapped candies into the machine that mixes and cools the chocolate!
糖果工厂最好要小心点了，如果他们尝试将包装好的糖果放入搅拌和冷却巧克力的机器。

### General Composition
### 通用组合

If we can define the composition of two functions, we can just keep going to support composing any number of functions. The general data visualization flow for any number of functions being composed looks like this:
如果我们能够定义两个函数的组合，我们也同样能够支持组合任意数量的函数。任意数目函数的组合的通用可视化数据流如下：

```
finalValue <-- func1 <-- func2 <-- ... <-- funcN <-- origValue
```

<p align="center">
	<img src="fig6.png" width="300">
</p>

Now the candy factory owns the best machine of all: a machine that can take any number of separate smaller machines and spit out a big fancy machine that does every step in order. That's one heck of a candy operation! It's Willy Wonka's dream!
现在糖果工厂拥有了最好的制造机：它能够接收任意数量独立的小机器，并吐出一个大只的、超赞的机器，能把每一步都按照顺序做好。这个糖果制作流程简直棒呆了！简直是威利·旺卡（译者注：《查理和巧克力工厂》中的人物，他拥有一座巧克力工厂）的梦想！

We can implement a general `compose(..)` utility like this:
我们能够像这样实现一个通用 `compose(..)` 工具函数：

```js
function compose(...fns) {
	return function composed(result){
		// copy the array of functions
		// 拷贝一份保存函数的数组
		var list = fns.slice();

		while (list.length > 0) {
			// take the last function off the end of the list
			// and execute it
			// 将最后一个函数从列表尾部拿出
			// 并执行它
			result = list.pop()( result );
		}

		return result;
	};
}

// or the ES6 => form
// ES6箭头函数形式写法
var compose =
	(...fns) =>
		result => {
			var list = fns.slice();

			while (list.length > 0) {
				// take the last function off the end of the list
				// and execute it
				// 将最后一个函数从列表尾部拿出
				// 并执行它
				result = list.pop()( result );
			}

			return result;
		};
```

Now let's look at an example of composing more than two functions. Recalling our `uniqueWords(..)` composition example, let's add a `skipShortWords(..)` to the mix:
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

Let's define `biggerWords(..)` that includes `skipShortWords(..)`. The manual composition equivalent we're looking for is `skipShortWords(unique(words(text)))`, so let's do it with `compose(..)`:
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

Now, let's recall `partialRight(..)` from Chapter 3 to do something more interesting with composition. We can build a right-partial application of `compose(..)` itself, pre-specifying the second and third arguments (`unique(..)` and `words(..)`, respectively); we'll call it `filterWords(..)` (see below).
现在，让我们回忆一下第 3 章中出现的 `partialRight(..)` 来让组合变的更有趣。我们能够构造一个由 `compose(..)` 自身组成的右偏函数应用，通过提前定义好第二和第三参数（`unique(..)` 和 `words(..)`）；我们把它称作 `filterWords(..)` （如下）。

Then, we can complete the composition multiple times by calling `filterWords(..)`, but with different first-arguments respectively:
然后，我们能够通过多次调用 `filterWords(..)` 来完成组合，但是每次的第一参数却各不相同。

```js
// Note: uses a `<= 4` check instead of the `> 4` check
// that `skipShortWords(..)` uses
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

Take a moment to consider what the right-partial application on `compose(..)` gives us. It allows us to specify ahead of time the first step(s) of a composition, and then create specialized variations of that composition with different subsequent steps (`biggerWords(..)` and `shorterWords(..)`). This is one of the most powerful tricks of FP!
花些时间考虑一下基于 `compose(..)` 的右偏函数应用给了我们什么。它允许我们在组合的第一步之前做指定，然后以不同后期步骤的组合来创建特定的变体。这是函数式编程中最强大的手段之一。

You can also `curry(..)` a composition instead of partial application, though because of right-to-left ordering, you might more often want to `curry( reverseArgs(compose), ..)` rather than just `curry( compose, ..)` itself.
你也能通过 `curry(..)` 作出的组合来替代偏函数应用，但因为从右往左的顺序，比起只使用 `curry( compose, ..)` 本身，你可能更想使用 `curry( reverseArgs(compose), ..)`。

**Note:** Since `curry(..)` (at least the way we implemented it in Chapter 3) relies on either detecting the arity (`length`) or having it manually specified, and `compose(..)` is a variadic function, you'll need to manually specify the intended arity like `curry(.. , 3)`.
**注意：** 因为 `curry(..)` 依赖于探测数目（`length`）或手动指定数目，而 `compose(..)` 是一个可变的函数，你需要手动指定数目，就像这样 `curry(.. , 3)`。

### Alternate Implementations
### 不同的实现

While you may very well never implement your own `compose(..)` to use in production, and rather just use a library's implementation as provided, I've found that understanding how it works under the covers actually helps solidify general FP concepts very well.
当然，你可能永远不会在生产中使用自己写的 `compose(..)`，而更倾向于使用某个库所提供的方案。但我发现了解底层工作的原理实际上对强化我们的函数式编程中通用概念非常有用。

So let's examine some different implementation options for `compose(..)`. We'll also see there are some pros/cons to each implementation, especially performance.
所以让我们看看对于 `compose(..)` 的不同实现方案。我们能看到每一种实现中有一些 pros/cons，特别是性能。

We'll be looking at the `reduce(..)` utility in detail later in the text, but for now, just know that it reduces a list (array) to a single finite value. It's like a fancy loop.
我们讲在文中稍后查看 `reduce(..)` 工具的细节，但现在，只需了解它将一个列表（数组）简化为一个单一的有限值。看起来像是一个很棒的循环体。

For example, if you did an addition-reduction across the list of numbers `[1,2,3,4,5,6]`, you'd be looping over them adding them together as you go. The reduction would add `1` to `2`, and add that result to `3`, and then add that result to `4`, and so on, resulting in the final summation: `21`.
举个栗子，如果在数字列表 `[1,2,3,4,5,6]` 上做加法约减，你将要循环它们，并且随着循环将它们加在一起。这一过程将首先将 `1` 加 `2`，然后将结果加 `3`，然后加 `4`，等等。最后得到总和：`21`。

The original version of `compose(..)` uses a loop and eagerly (aka, immediately) calculates the result of one call to pass into the next call. We can do that same thing with `reduce(..)`:
原始版本的 `compose(..)` 使用一个循环并且饥渴的（也就是，立刻）执行计算，从一个调用的结果传递到下一个调用。我们可以通过 `reduce(..)` （代替循环）做到同样的事。

```js
function compose(...fns) {
	return function composed(result){
		return fns.reverse().reduce( function reducer(result,fn){
			return fn( result );
		}, result );
	};
}

// or the ES6 => form
// ES6箭头函数形式写法
var compose = (...fns) =>
	result =>
		fns.reverse().reduce(
			(result,fn) =>
				fn( result )
			, result
		);
```

Notice that the `reduce(..)` looping happens each time the final `composed(..)` function is run, and that each intermediate `result(..)` is passed along to the next iteration as the input to the next call.
注意到 `reduce(..)` 循环发生在最后的 `composed(..)` 运行时，并且每一个中间的 `result(..)` 将会在下一次调用时作为输入值传递给下一个迭代。

The advantage of this implementation is that the code is more concise and also that it uses a well-known FP construct: `reduce(..)`. And the performance of this implementation is also similar to the original `for`-loop version.
这种实现的优点就是代码更简练，并且使用了常见的函数式编程结构：`reduce(..)`。这种实现方式的性能和原始的 `for`-循环版本很相近。 

However, this implementation is limited in that the outer composed function (aka, the first function in the composition) can only receive a single argument. Most other implementations pass along all arguments to that first call. If every function in the composition is unary, this is no big deal. But if you need to pass multiple arguments to that first call, you'd want a different implementation.
但是，这种实现局限处在于外层的组合函数（也就是，组合中的第一个函数）只能接收一个参数。其他大多数实现在首次调用的时候就把所有参数传进去了。如果组合中的每一个函数都是一元的，这个方案没啥大问题。但如果你需要给第一个调用传递多参数，那么你可能需要不同的实现方案。

To fix that first call single-argument limitation, we can still use `reduce(..)` but produce a lazy-evaluation function wrapping:
为了修正第一次调用的单参数限制，我们可以仍使用 `reduce(..)` 但加一个懒执行函数包裹器：

```js
function compose(...fns) {
	return fns.reverse().reduce( function reducer(fn1,fn2){
		return function composed(...args){
			return fn2( fn1( ...args ) );
		};
	} );
}

// or the ES6 => form
var compose =
	(...fns) =>
		fns.reverse().reduce( (fn1,fn2) =>
			(...args) =>
				fn2( fn1( ...args ) )
		);
```

Notice that we return the result of the `reduce(..)` call directly, which is itself a function, not a computed result. *That* function lets us pass in as many arguments as we want, passing them all down the line to the first function call in the composition, then bubbling up each result through each subsequent call.

Instead of calculating the running result and passing it along as the `reduce(..)` looping proceeds, this implementation runs the `reduce(..)` looping **once** up front at composition time, and defers all the function call calculations -- referred to as lazy calculation. Each partial result of the reduction is a successively more wrapped function.

When you call the final composed function and provide one or more arguments, all the levels of the big nested function, from the inner most call to the outer, are executed in reverse succession (not via a loop).

The performance characteristics will potentially be different than in the previous `reduce(..)`-based implementation. Here, `reduce(..)` only runs once to produce a big composed function, and then this composed function call simply executes all its nested functions each call. In the former version, `reduce(..)` would be run for every call.

Your mileage may vary on which implementation is better, but keep in mind that this latter implementation isn't limited in argument count the way the former one is.

We could also define `compose(..)` using recursion. The recursive definition for `compose(fn1,fn2, .. fnN)` would look like:

```
compose( compose(fn1,fn2, .. fnN-1), fnN );
```

**Note:** We will cover recursion in deep detail in Chapter 9, so if this approach seems confusing, feel free to skip it for now and come back later after reading that chapter.

Here's how we implement	`compose(..)` with recursion:

```js
function compose(...fns) {
	// pull off the last two arguments
	var [ fn1, fn2, ...rest ] = fns.reverse();

	var composedFn = function composed(...args){
		return fn2( fn1( ...args ) );
	};

	if (rest.length == 0) return composedFn;

	return compose( ...rest.reverse(), composedFn );
}

// or the ES6 => form
var compose =
	(...fns) => {
		// pull off the last two arguments
		var [ fn1, fn2, ...rest ] = fns.reverse();

		var composedFn =
			(...args) =>
				fn2( fn1( ...args ) );

		if (rest.length == 0) return composedFn;

		return compose( ...rest.reverse(), composedFn );
	};
```

I think the benefit of a recursive implementation is mostly conceptual. I personally find it much easier to think about a repetitive action in recursive terms instead of in a loop where I have to track the running result, so I prefer the code to express it that way.

Others will find the recursive approach quite a bit more daunting to mentally juggle. I invite you to make your own evaluations.

## Reordered Composition

We talked earlier about the right-to-left ordering of standard `compose(..)` implementations. The advantage is in listing the arguments (functions) in the same order they'd appear if doing the composition manually.

The disadvantage is they're listed in the reverse order that they execute, which could be confusing. It was also more awkward to have to use `partialRight(compose, ..)` to pre-specify the *first* function(s) to execute in the composition.

The reverse ordering, composing from left-to-right, has a common name: `pipe(..)`. This name is said to come from Unix/Linux land, where multiple programs are strung together by "pipe"ing (`|` operator) the output of the first one in as the input of the second, and so on (i.e., `ls -la | grep "foo" | less`).

`pipe(..)` is identical to `compose(..)` except it processes through the list of functions in left-to-right order:

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

In fact, we could just define `pipe(..)` as the arguments-reversal of `compose(..)`:

```js
var pipe = reverseArgs( compose );
```

That was easy!

Recall this example from general composition earlier:

```js
var biggerWords = compose( skipShortWords, unique, words );
```

To express that with `pipe(..)`, we just reverse the order we list them in:

```js
var biggerWords = pipe( words, unique, skipShortWords );
```

The advantage of `pipe(..)` is that it lists the functions in order of execution, which can sometimes reduce reader confusion. It may be simpler to see `pipe(words,unique,skipShortWords)` and read that we do `words(..)` first, then `unique(..)`, and finally `skipShortWords(..)`.

`pipe(..)` is also handy if you're in a situation where you want to partially apply the *first* function(s) that execute. Earlier we did that with right-partial application of `compose(..)`.

Compare:

```js
var filterWords = partialRight( compose, unique, words );

// vs

var filterWords = partial( pipe, words, unique );
```

As you may recall from `partialRight(..)`'s definition in Chapter 3, it uses `reverseArgs(..)` under the covers, just as our `pipe(..)` now does. So we get the same result either way.

The slight performance advantage to `pipe(..)` in this specific case is that since we're not trying to preserve the right-to-left argument order of `compose(..)` by doing a right-partial application, using `pipe(..)` we don't need to reverse the argument order back, like we do inside `partialRight(..)`. So `partial(pipe, ..)` is a little better here than `partialRight(compose, ..)`.

In general, `pipe(..)` and `compose(..)` will not have any significant performance differences when using a well-established FP library.

## Abstraction

Abstraction is often defined as pulling out the generality between two or more tasks. The general part is defined once, so as to avoid repetition. To perform each task's specialization, the general part is parameterized.

For example, consider this (obviously contrived) code:

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

Both of these utilities are storing a value in a data source. That's the generality. The specialty is that one of them sticks the value at the end of an array, while the other sets the value at a property name of an object.

So let's abstract:

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

The general task of referencing a property on an object (or array, thanks to JS's convenient operator overloading of `[ ]`) and setting its value is abstracted into its own function `storeData(..)`. While this utility only has a single line of code right now, one could envision other general behavior that was common across both tasks, such as generating a unique numeric ID or storing a timestamp with the value.

If we repeat the common general behavior in multiple places, we run the maintenance risk of changing some instances but forgetting to change others. There's a principle at play in this kind of abstraction, often referred to as DRY (don't repeat yourself).

DRY strives to have only one definition in a program for any given task. An alternate quip to motivate DRY coding is that programmers are just generally lazy and don't want to do unnecessary work.

Abstraction can be taken too far. Consider:

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

In an effort to DRY and avoid repeating an `if` statement, we moved the conditional into the general abstraction. We also assumed that we may have checks for non-empty strings or non-`undefined` values elsewhere in the program, so we might as well DRY those out, too!

This code *is* more DRY, but to an overkill extent. Programmers must be careful to apply the appropriate levels of abstraction to each part of their program, no more, no less.

Regarding our greater discussion of function composition in this chapter, it might seem like its benefit is this kind of DRY abstraction. But let's not jump to that conclusion, because I think composition actually serves a more important purpose in our code.

Moreover, **composition is helpful even if there's only one occurrence of something** (no repetition to DRY out).

Aside from generalization vs specialization, I think there's another more useful definition for abstraction, as revealed by this quote:

> ... abstraction is a process by which the programmer associates a name with a potentially complicated program fragment, which can then be thought of in terms of its purpose of function, rather than in terms of how that function is achieved. By hiding irrelevant details, abstraction reduces conceptual complexity, making it possible for the programmer to focus on a manageable subset of the program text at any particular time.
>
> Programming Language Pragmatics, Michael L Scott
>
> https://books.google.com/books?id=jM-cBAAAQBAJ&pg=PA115&lpg=PA115&dq=%22making+it+possible+for+the+programmer+to+focus+on+a+manageable+subset%22&source=bl&ots=yrJ3a-Tvi6&sig=XZwYoWwbQxP2w5qh2k2uMAPj47k&hl=en&sa=X&ved=0ahUKEwjKr-Ty35DSAhUJ4mMKHbPrAUUQ6AEIIzAA#v=onepage&q=%22making%20it%20possible%20for%20the%20programmer%20to%20focus%20on%20a%20manageable%20subset%22&f=false

// TODO: make a proper reference to this book/quote, or at least find a better online link

The point this quote makes is that abstraction -- generally, pulling out some piece of code into its own function -- serves the primary purpose of separating apart two pieces of functionality so that each piece can be focused on independent of the other.

Note that abstraction in this sense is not intended to *hide* details, as if to treat things as black boxes. That notion is more closely associated with the programming principle of encapsulation. **We're not abstracting to hide, but to separate to improve focus**.

Recall that at the outset of this text I described the goal of FP as creating more readable, understandable code. One effective way of doing that is untangling complected -- read: tightly braided, as in strands of rope -- code into separate, simpler -- read: loosely bound -- pieces of code. In that way, the reader isn't distracted by the details of one part while looking for the details of the other part.

Our higher goal is not to implement something only once, as it is with the DRY mindset. As a matter of fact, sometimes we'll actually repeat ourselves in code. Instead, we seek to implement separate things, separately. We're trying to improve focus, because that improves readability.

Another way of describing this goal is with imperative vs declarative programming style. Imperative code is primarily concerned with explicitly stating *how* to accomplish a task. Declarative code states *what* the outcome should be, and leaves the implementation to some other responsibility.

In other words, declarative code abstracts the *what* from the *how*. Typically declarative coding is favored in readability over imperative, though no program (except of course machine code 1's and 0's) is ever entirely one or the other. The programmer must seek balance between them.

ES6 added many syntactic affordances that transform old imperative operations into newer declarative forms. Perhaps one of the clearest is destructuring. Destructuring is a pattern for assignment that describes how a compound value (object, array) is taken apart into its constituent values.

Here's an example of array destructuring:

```js
function getData() {
	return [1,2,3,4,5];
}

// imperative
var tmp = getData();
var a = tmp[0];
var b = tmp[3];

// declarative
var [ a ,,, b ] = getData();
```

The *what* is assigning the first value of the array to `a` and the fourth value to `b`. The *how* is getting a reference to the array (`tmp`) and manually referencing indexes `0` and `3` in assignments to `a` and `b`, respectively.

Does the array destructuring *hide* the assignment? Depends on your perspective. I'm asserting that it simply separates the *what* from the *how*. The JS engine still does the assignments, but it prevents you from having to be distracted by *how* it's done.

Instead, you read `[ a ,,, b ] = ..` and can see the assignment pattern merely telling you *what* will happen. Array destructuring is an example of declarative abstraction.

### Composition As Abstraction

What's all this have to do with function composition? Function composition is also declarative abstraction.

Recall the `shorterWords(..)` example from earlier. Let's compare an imperative and declarative definition for it:

```js
// imperative
function shorterWords(text) {
	return skipLongWords( unique( words( text ) ) );
}

// declarative
var shorterWords = compose( skipLongWords, unique, words );
```

The declarative form focuses on the *what* -- these 3 functions pipe data from a string to a list of shorter words -- and leaves the *how* to the internals of `compose(..)`.

In a bigger sense, the `shorterWords = compose(..)` line explains the *how* for defining a `shorterWords(..)` utility, leaving this declarative line somewhere else in the code to focus only on the *what*:

```js
shorterWords( text );
```

Composition abstracts getting a list of shorter words from the steps it takes to do that.

By contrast, what if we hadn't used composition abstraction?

```js
var wordsFound = words( text );
var uniqueWordsFound = unique( wordsFound );
skipLongWords( uniqueWordsFound );
```

Or even:

```js
skipLongWords( unique( words( text ) ) );
```

Either of these two versions demonstrates a more imperative style as opposed to the prior declarative style. The reader's focus in those two snippets is inextricably tied to the *how* and less on the *what*.

Function composition isn't just about saving code with DRY. Even if the usage of `shorterWords(..)` only occurs in one place -- so there's no repetition to avoid! -- separating the *how* from the *what* still improves our code.

Composition is a powerful tool for abstraction that transforms imperative code into more readable declarative code.

## Revisiting Points

Now that we've thoroughly covered composition -- a trick that will be immensely helpful in many areas of FP -- let's watch it in action by revisiting point-free style from "No Points" in Chapter 3 with a scenario that's a fair bit more complex to refactor:

```js
// given: ajax( url, data, cb )

var getPerson = partial( ajax, "http://some.api/person" );
var getLastOrder = partial( ajax, "http://some.api/order", { id: -1 } );

getLastOrder( function orderFound(order){
	getPerson( { id: order.personId }, function personFound(person){
		output( person.name );
	} );
} );
```

The "points" we'd like to remove are the `order` and `person` parameter references.

Let's start by trying to get the `person` "point" out of the `personFound(..)` function. To do so, let's first to define:

```js
function extractName(person) {
	return person.name;
}
```

But let's observe that this operation could instead be expressed in generic terms: extracting any property by name off of any object. Let's call such a utility `prop(..)`:

```js
function prop(name,obj) {
	return obj[name];
}

// or the ES6 => form
var prop =
	(name,obj) =>
		obj[name];
```

While we're dealing with object properties, let's also define the opposite utility: `setProp(..)` for setting a property value onto an object.

However, we want to be careful not to just mutate an existing object but rather create a clone of the object to make the change to, and then return it. The reasons for such care will be discussed in detail in Chapter 5.

```js
function setProp(name,obj,val) {
	var o = Object.assign( {}, obj );
	o[name] = val;
	return o;
}
```

Now, to define an `extractName(..)` that pulls a `"name"` property off an object, we'll partially apply `prop(..)`:

```
var extractName = partial( prop, "name" );
```

**Note:** Don't miss that `extractName(..)` here hasn't actually extracted anything yet. We partially applied `prop(..)` to make a function that's waiting to extract the `"name"` property from whatever object we pass into it. We could also have done it with `curry(prop)("name")`.

Next, let's narrow the focus on our example's nested lookup calls to this:

```js
getLastOrder( function orderFound(order){
	getPerson( { id: order.personId }, outputPersonName );
} );
```

How can we define `outputPersonName(..)`? To visualize what we need, think about the desired flow of data:

```
output <-- extractName <-- person
```

`outputPersonName(..)` needs to be a function that takes an (object) value, passes it into `extractName(..)`, then passes that value to `output(..)`.

Hopefully you recognized that as a `compose(..)` operation. So we can define `outputPersonName(..)` as:

```
var outputPersonName = compose( output, extractName );
```

The `outputPersonName(..)` function we just created is the callback provided to `getPerson(..)`. So we can define a function called `processPerson(..)` that presets the callback argument, using `partialRight(..)`:

```js
var processPerson = partialRight( getPerson, outputPersonName );
```

Let's reconstruct the nested lookups example again with our new function:

```js
getLastOrder( function orderFound(order){
	processPerson( { id: order.personId } );
} );
```

Phew, we're making good progress!

But we need to keep going and remove the `order` "point". The next step is to observe that `personId` can be extracted from an object (like `order`) via `prop(..)`, just like we did with `name` on the `person` object:

```js
var extractPersonId = partial( prop, "personId" );
```

To construct the object (of the form `{ id: .. }`) that needs to be passed to `processPerson(..)`, let's make another utility for wrapping a value in an object at a specified property name, called `makeObjProp(..)`:

```js
function makeObjProp(name,value) {
	return setProp( name, {}, value );
}

// or the ES6 => form
var makeObjProp =
	(name,value) =>
		setProp( name, {}, value );
```

**Tip:** This utility is known as `objOf(..)` in the Ramda library.

Just as we did with `prop(..)` to make `extractName(..)`, we'll partially apply `makeObjProp(..)` to build a function `personData(..)` that makes our data object:

```js
var personData = partial( makeObjProp, "id" );
```

To use `processPerson(..)` to perform the lookup of a person attached to an `order` value, the conceptual flow of data through operations we need is:

```
processPerson <-- personData <-- extractPersonId <-- order
```

So we'll just use `compose(..)` again to define a `lookupPerson(..)` utility:

```js
var lookupPerson = compose( processPerson, personData, extractPersonId );
```

And... that's it! Putting the whole example back together without any "points":

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

Wow. Point-free. And `compose(..)` turned out to be really helpful in two places!

I think in this case, even though the steps to derive our final answer were a bit drawn out, the end result is much more readable code, because we've ended up explicitly calling out each step.

And even if you didn't like seeing/naming all those intermediate steps, you can preserve point-free but wire the expressions together without individual variables:

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

This snippet is less verbose for sure, but I think it's less readable than the previous snippet where each operation is its own variable. Either way, composition helped us with our point-free style.

## Summary

Function composition is a pattern for defining a function that routes the output of one function call into another function call, and its output to another, and so on.

Because JS functions can only return single values, the pattern essentially dictates that all functions in the composition (except perhaps the first called) need to be unary, taking only a single input from the output of the previous.

Instead of listing out each step as a discrete call in our code, function composition using a utility like `compose(..)` abstracts that implementation detail so the code is more readable, allowing us to focus on *what* the composition will be used to accomplish, not *how* it will be performed.

Composition -- declarative data flow -- is one of the most important tools that underpins most of the rest of FP.
