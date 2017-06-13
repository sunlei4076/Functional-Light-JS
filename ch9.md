# Javascript轻量级函数式编程
# 第9章：递归

下面，我们将进入到递归的论题。

<hr>

*(部分页面故意留白)*

<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>

<div style="page-break-after: always;"></div>

我们来谈谈递归吧。在我们下水之前，请查阅上一页的正式定义。

我知道，这个笑话弱爆了 :)

大部分的开发人员都承认递归是一门非常强大的编程技术，但他们并不喜欢去使用它。在这个意义上，我把它放在与正则表达式相同的类别中。强大但又令人困惑，因此被视为 *不值得的努力*。

我是递归编程的超级粉丝，你，也可以的！在这一章节中我的目标就是说服你：递归是一个重要的工具，你应该带着它在你的函数式编程中编码。当正确使用时，递归编程可以轻松的描述复杂问题。

## 定义

所谓递归，是当一个函数调用自身，并且该调用做了同样的事情，这个循环持续到基本条件满足时，调用循环返回。

**警告：** 如果你不能确保基本条件是递归的 *终结者* ，递归将会一直执行下去，并且会把你的项目损坏或锁死；基本条件没毛病是非常重要的！

但是... 这个书面形式的定义太乱了。我们可以做的更好些。思考下这个递归函数：

```js
function foo(x) {
	if (x < 5) return x;
	return foo( x / 2 );
}
```

设想一下，如果我们调用 `foo(16)` 将会发生什么：

<p align="center">
	<img src="fig13.png" width="850">
</p>

在 step 2 中, `x / 2` 的结果是 `8`， 这个结果以参数的形式传递到 `foo(..)` 并运行。同样的，在 step 3 中， `x / 2` 的结果是 `4`，这个结果以参数的形式传递到另一个 `foo(..)` 并运行。希望这部分内容够简单。

但是一些人经常会在 step 4 中卡壳。一旦我们满足了基本条件 `x` (值为4) 是 `<5` 的，我们将不再调用递归函数，只是(有效的)执行了 `return 4`。
特别是图中返回 `4` 的虚线那块，它简化了那里的过程，因此我们来深入了解最后一步，并把它折分为三个子步骤：

<p align="center">
	<img src="fig14.png" width="850">
</p>

一旦满足了基本条件，返回值将会通过一堆的回调并逐级返回到其相应的回调函数中，而最终的一次 `return` 就是最后的结果。

另外一个递归实例：

```js
function isPrime(num,divisor = 2){
	if (num < 2 || (num > 2 && num % divisor == 0)) {
		return false;
	}
	if (divisor <= Math.sqrt( num )) {
		return isPrime( num, divisor + 1 );
	}

	return true;
}
```
这个质数的判断主要是通过验证，从2到 `num` 的平方根之间的每个整数，看是否存在某一整数可以整除 `num` (`%` 求余结果为 `0`)。如果存在这样的整数，那么 `num` 不是质数。反之，是质数。`divisor + 1` 使用递归来遍历每个可能的 `divisor` 值。

递归的最着名的例子之一是计算斐波那契数，它的数列如下所示：

```
fib( 0 ): 0
fib( 1 ): 1
fib( n ):
	fib( n - 2 ) + fib( n - 1 )
```

**注意:** 数列的前几个数值是： 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ... 每一个数字都是数列中前两个数字之和。

直接用代码来定义斐波那契：

```js
function fib(n) {
	if (n <= 1) return n;
	return fib( n - 2 ) + fib( n - 1 );
}
```

函数 `fib(..)` 对自身进行了两次递归调用，这通常叫作二分递归查找。后面我们将会更多的讨论二分递归查找。

在整个章节中，我们将会用不同形式的 `fib(..)` 来说明关于递归的想法，但不太好的地方就是，这种特殊的方式会造成很多重复性的工作。 `fib(n-1)` 和 `fib(n-2)` 运行时候两者之间并没有任何的共享，却几乎又完全相互重叠，直到整个整数空间(译者注：形参 `n`)降到 `0` 。

在第五章的性能优化方面我们简单的谈到了记忆存储技术。本章中，记忆存储技术使得任意一个传入到 `fib(..)` 的数值只会被计算一次而不是多次。虽然我们不会在这里过多的讨论这个技术话题，但不论是递归或其它任何算法，我们都要谨记，性能优化是非常重要的。

### 相互递归

当一个函数调用自身时，很明显，这叫作直接递归。比如前面部分我们谈到的 `foo(..)`，`isPrime(..)`以及 `fib(..)`。如果在一个递归循环中，出现两个及以上的函数相互调用，则称之为相互递归。

这两个函数就是相互递归：

```js
function isOdd(v) {
	if (v === 0) return false;
	return isEven( Math.abs( v ) - 1 );
}

function isEven(v) {
	if (v === 0) return true;
	return isOdd( Math.abs( v ) - 1 );
}
```

是的，这个奇偶数的判断笨笨的。但也给我们提供了一些思路：某些算法可以根据相互递归来定义。

回顾下上节中的二分递归法 `fib(..)`；我们可以换成相互递归来表示：

```js
function fib_(n) {
	if (n == 1) return 1;
	else return fib( n - 2 );
}

function fib(n) {
	if (n == 0) return 0;
	else return fib( n - 1 ) + fib_( n );
}
```

**注意：** `fib(..)` 相互递归的实现方式改编自 "用相互递归来实现斐波纳契数列" 研究报告(https://www.researchgate.net/publication/246180510_Fibonacci_Numbers_Using_Mutual_Recursion)。

虽然这些相互递归的示例有点不切实际，但是在更复杂的使用场景下，相互递归是非常有用的。

### 为什么选择递归？

现在我们已经给出了递归的定义和说明，下面来看下，为什么说递归是有用的。

递归深谙函数式编程之精髓，最被广泛引证的原因是，在回调堆栈中，递归把(大部分)显式状态跟踪换为了隐式状态。通常，当问题需要条件分支和回溯计算，这时候递归非常有用，并且在纯迭代环境中管理这种状态，是相当棘手的；最起码，这些代码是不可或缺且晦涩难懂。但是在堆栈上调用每一级的分支作为其自己的作用域，很明显，这通常会影响到代码的可读性。

简单的迭代算法可以用递归来表达：

```js
function sum(total,...nums) {
	for (let i = 0; i < nums.length; i++) {
		total = total + nums[i];
	}

	return total;
}

// vs

function sum(num1,...nums) {
	if (nums.length == 0) return num1;
	return num1 + sum( ...nums );
}
```

不仅仅是为了消除回调栈中的 `for` 循环，而是用 `return`s 的形式在回调栈中隐式的跟踪增量求和（ `total` 的间歇状态），而并非在每次迭代中重新分配 `total`。通常情况下，只要存在可能，函数式编程的程序员更倾向于避免重新分配局部变量。

在我们总结的这种基本算法里面，这些差异是微乎其微的。但是，随着算法复杂度的提升，您很可能会看到递归带来的效果而不是命令状态跟踪。
在像这种总结的基本算法中，这种差异是微不足道的。 但是，您的算法越复杂，越有可能看到递归的回报而不是命令状态跟踪。

## 声明式递归

数学家使用 **Σ** 符号来表示一列数字的总和。他们这么做的主要原因是，如果他们正在使用更复杂的公式且必须要手动写出求和的话，这是非常麻烦（且阅读性非常差！），比如 
 `1 + 3 + 5 + 7 + 9 + ..`。符号是数学的声明式语言！

递归是声明式的算法，就好比 **Σ** 是数学的声明一样。递归说明：一个问题存在解决方案，但并不一定要求阅读代码的人了解该解决方案的工作原理。我们来思考下，找出入参最大偶数值的两种方法：

```js
function maxEven(...nums) {
	var num = -Infinity;

	for (let i = 0; i < nums.length; i++) {
		if (nums[i] % 2 == 0 && nums[i] > num) {
			num = nums[i];
		}
	}

	if (num !== -Infinity) {
		return num;
	}
}
```

这种实现方式并不是特别难处理，但它的一些细微的问题也不容忽视。很明显，运行 `maxEven()`，`maxEven(1)` 和 `maxEven(1,13)` 都将会返回 `undefined`？最终的 `if` 表达式是必需的吗？

我们试着换一个递归的方法来对比下。我们可以通过这种方式来表示递归：

```
maxEven( nums ):
	maxEven( nums.0, maxEven( ...nums.1 ) )
```

换句话说，我们可以将数表的 max-even 定义为其余数字的 max-even 与第一个数字的 max-even 的结果。例如：

```
maxEven( 1, 10, 3, 2 ):
	maxEven( 1, maxEven( 10, maxEven( 3, maxEven( 2 ) ) )
```

在JS中实现这个递归定义的方法之一是：

```js
function maxEven(num1,...restNums) {
	var maxRest = restNums.length > 0 ?
			maxEven( ...restNums ) :
			undefined;

	return (num1 % 2 != 0 || num1 < maxRest) ?
		maxRest :
		num1;
}
```

那么这个方法有什么优点吗？

首先，参数与之前不一样了。我专门把第一个参数叫作 `num1`，剩余的其它参数放在一起叫作 `restNums`。我们本可以把所有参数都放在 `nums` 数组中，并从 `nums[0]` 获取第一个参数。为什么呢？

函数的参数是专门为递归定义的。 它看起来像这样：

```
maxEven( num1, ...restNums ):
	maxEven( num1, maxEven( ...restNums ) )
```

你有发现参数和递归之间的对称性吗？

当我们在函数体的参数中进一步提升递归的定义，函数的声明也会得到提升。如果我们能够把递归的定义从参数反映到函数体中，那就更棒了。

但我想说最明显的改进是，`for` 循环造成的错乱感没有了。所有循环逻辑都被抽象为递归回调栈，所以这些东西不会造成代码混乱。我们可以轻松的把精力集中在一次比较两个数字来找到最大偶数值的逻辑中 -- 不管怎么说，这都是很重要的部分！

从心理上来讲，这如同一位数学家在更庞大的方程中使用 **Σ** 求和一样。我们说，“数列中剩余值的最大偶数是通过 `maxEven(...restNums)` 计算出来的，所以我们只需要继续推断这一部分。”

另外，我们用 `restNums.length > 0` 保证推断更加合理，因为当没有参数的情况下，返回的 `maxRest` 结果肯定是 `undefined`。我们不需要对这部分的推理投入额外的精力。这个基本条件（没有参数情况下）显而易见。

接下来，我们把精力放在对比 `num1` 和 `maxRest` 上 -- 算法的主要逻辑是如何确定两个数字中的哪一个（如果有的话）是最大偶数。如果 `num1` 不是偶数（`num1 % 2 != 0`），或着它小于 `maxRest`，那么 `maxRest` 会 `return` 掉，即使 `maxRest` 的值是 `undefined`。否则，返回结果会是 `num1`。

在阅读整个实现过程中，与命令式的方法相比，我所做这个例子的推理过程更加直接，核心点更加突出，少做无用功；比 `for`-循环中引用 `无穷数值` 这一方法 **更具有声明性**。 

**小贴士：** 我们应该指出，除了手动迭代或递归之外，另一种（可能更好的）建模的方法是我们在在第7章中讨论的列表操作。我们先把数列中的偶数用 `filter(..)` 过滤出来，然后通过递归 `reduce(..)` 函数（对比两个数值并返回其中较大的数值）来找到最大值。在这里，我们只是使用这个例子来说明在手动迭代中递归的声明性更强。

还有一个递归的例子：计算二叉树的深度。二叉树的深度是指通过树的节点向下（左或右）的最长路径。二叉树深度的另外一种定义是递归：任何树节点的深度为1（当前节点）加上来自其左侧或右侧子树的深度的最大值：

```
depth( node ):
	1 + max( depth( node.left ), depth( node.right ) )
```

直接转换为二分法递归函数：

```js
function depth(node) {
	if (node) {
		let depthLeft = depth( node.left );
		let depthRight = depth( node.right );
		return 1 + max( depthLeft, depthRight );
	}

	return 0;
}
```

我不打算列出这个算法的命令式形式，但请相信我，它太麻烦、过于命令式了。这种递归方法很不错，声明也很优雅。它遵循递归的定义，与递归定义的算法非常接近，省心。

并不是所有的问题都是完全可递归的。这不是你应该尝试广泛应用的一些新技术。但是递归可以非常有效地将问题的表达，从更具必要性转变为更有声明性。

## 栈、堆

一起看下之前的两个递归函数 `isOdd(..)` 和 `isEven(..)`：

```js
function isOdd(v) {
	if (v === 0) return false;
	return isEven( Math.abs( v ) - 1 );
}

function isEven(v) {
	if (v === 0) return true;
	return isOdd( Math.abs( v ) - 1 );
}
```

如果你执行下面这行代码，在大多数浏览器里面都会报错：

```js
isOdd( 33333 );			// RangeError: Maximum call stack size exceeded
```

这个错误是神马情况？引擎抛出这个错误，是因为它试图保护系统内存不会被你的程序吃掉。为了解释这个问题，我们需要先看看当函数调用时JS引擎中发生了什么。

每个函数调用都将开辟出一小块称为堆栈桢的内存。堆栈桢中包含了函数语句当前状态的某些重要信息，包括任意变量的值。之所以这样，是因为一个函数暂停去执行另外一个函数，而另外一个函数运行结束后，引擎需要返回到之前暂停时候的状态继续执行。

当第二个函数开始执行，堆栈桢增加到 2 个。如果第二个函数又调用了另外一个函数，堆栈桢将增加到 3 个，以此类推。“堆、栈”的意思是，函数被它前一个函数调用时，这个函数桢会被“推”到最顶部。当这个函数调用结束后，它的桢会从堆栈中退出。

看下这段程序：

```js
function foo() {
	var z = "foo!";
}

function bar() {
	var y = "bar!";
	foo();
}

function baz() {
	var x = "baz!";
	bar();
}

baz();
```

来一步步想象下这个程序的堆栈桢：

<p align="center">
	<img src="fig15.png" width="600">
</p>

**注意：** 如果这些函数间没有相互调用，而只是依次执行 -- 比如前一个函数运行结束后才开始调用下一个函数 `baz(); bar(); foo();` -- 则堆栈桢并没有产生；因为在下一个函数开始之前，上一个函数运行结束并把它的桢从堆栈里面移除了。

所以，每一个函数运行时候，都会占用一些内存。对多数程序来说，这没什么大不了的，不是吗？但是，一旦你引用了递归，就有什么大不了咯。
虽然你几乎肯定不会在一个调用堆栈中手动调用成千（或数百）次不同的函数，但你很容易看到产生数万个或更多递归调用的堆栈。

当引擎认为回调堆栈增加的太多并且应该停止增加时候，它会以任意的限制来阻止当前步骤，所以 `isOdd(..)` 或 `isEven(..)` 函数抛出了 `RangeError` 未知错误。这不太可能是内存接近零时候产生的限制，而是引擎的预测，因为如果这种程序持续运行下去，内存会爆掉的。由于引擎无法判断一个程序最终是否会停止，所以它必须做出确定的猜测。

引擎的限制因情况而定。规范里面并没有任何说明，因此，它也不是 *必需的*。但如果没有限制的话，设备很容易遭到破坏或恶意代码攻击，故而几乎所有的JS引擎都有一个限制。不同的设备环境、不同的引擎，会有不同的限制，也就无法预测或保证函数回调堆栈能调用多少次。

在处理大数据量时候，这个限制对于开发人员来说，会对递归的性能有一定的要求。我认为，这种限制也可能是造成开发人员不喜欢使用递归编程的最大原因。
遗憾的是，递归编程是一种编程思想而不是主流的编程技术。

### 尾调用

Recursion far predates JS, and so do these memory limitations. Back in the 1960s, developers were wanting to use recursion and running up against hard limits of device memory of their powerful computers that were far lower than we have on our watches today.

Fortunately, a powerful observation was made in those early days that still offers hope. The technique is called *tail calls*.

The idea is that if a call from function `baz()` to function `bar()` happens at the very end of function `baz()`'s execution -- referred to as a tail call -- the stack frame for `baz()` isn't needed anymore. That means that either the memory can be reclaimed, or even better, simply reused to handle function `bar()`'s execution. Visualizing:

<p align="center">
	<img src="fig16.png" width="600">
</p>

Tail calls are not really directly related to recursion, per se; this notion holds for any function call. But your manual non-recursion call stacks are unlikely to go beyond maybe 10 levels deep in most cases, so the chances of tail calls impacting your program's memory footprint are pretty low.

Tail calls really shine in the recursion case, because it means that a recursive stack could run "forever", and the only performance concern would be computation, not fixed memory limitations. Tail call recursion can run in `O(1)` fixed memory usage.

These sorts of techniques are often referred to as Tail Call Optimizations (TCO), but it's important to distinguish the ability to detect a tail call to run in fixed memory space, from the techniques that optimize this approach. Technically, tail calls themselves are not a performance optimization as most people would think, as they might actually run slower than normal calls. TCO is about optimizing tail calls to run more efficiently.

### Proper Tail Calls (PTC)

JavaScript has never required (nor forbidden) tail calls, until ES6. ES6 mandates recognition of tail calls, of a specific form referred to as Proper Tail Calls (PTC), and the guarantee that code in PTC form will run without unbounded stack memory growth. Practically speaking, this means we should not get `RangeError`s thrown if we adhere to PTC.

First, PTC in JavaScript requires strict mode. You should already be using strict mode, but if you aren't, this is yet another reason you should already be using strict mode. Did I mention, yet, you should already be using strict mode!?

Second, a *proper* tail call is exactly like this:

```js
return foo( .. );
```

In other words, the function call is the last thing to execute in the function, and whatever value it returns is explicitly `return`ed. In this way, JS can be absolutely guaranteed that the current stack frame won't be needed anymore.

These *are not* PTC:

```js
foo();
return;

// or

var x = foo( .. );
return x;

// or

return 1 + foo( .. );
```

**Note:** A JS engine *could* do some code reorganization to realize that `var x = foo(); return x;` is effectively the same as `return foo();`, which would then make it eligible as PTC. But that is not be required by the specification.

The `1 +` part is definitely processed *after* `foo(..)` finishes, so the stack frame has to be kept around.

However, this *is* PTC:

```js
return x ? foo( .. ) : bar( .. );
```

After the `x` condition is computed, either `foo(..)` or `bar(..)` will run, and in either case, the return value will be always be `return`ed back. That's PTC form.

Binary recursion -- two (or more!) recursive calls are made -- can never be effective PTC as-is, because all the recursion has to be in tail call position to avoid the stack growth. Earlier, we showed an example of refactoring from binary recursion to mutual recursion. It may be possible to achieve PTC from a multiple-recursive algorithm by splitting each into separate function calls, where each is expressed respectively in PTC form.

## Rearranging Recursion

If you want to use recursion but your problem set could grow enough eventually to exceed the stack limit of the JS engine, you're going to need to rearrange your recursive calls to take advantage of PTC (or avoid nested calls entirely). There are several refactoring strategies that can help, but there are of course tradeoffs to be aware of.

As a word of caution, always keep in mind that code readability is our overall most important goal. If recursion along with some combination of these following strategies results in harder to read/understand code, **don't use recursion**; find another more readable approach.

### Replacing The Stack

The main problem with recursion is its memory usage, keeping around the stack frames to track the state of a function call while it dispatches to the next recursive call iteration. If we can figure out how to rearrange our usage of recursion so that the stack frame doesn't need to be kept, then we can express recursion with PTC and take advantage of the JS engine's optimized handling of tail calls.

Let's recall the summation example from earlier:

```js
function sum(num1,...nums) {
	if (nums.length == 0) return num1;
	return num1 + sum( ...nums );
}
```

This isn't in PTC form because after the recursive call to `sum(...nums)` is finished, the `total` variable is added to that result. So, the stack frame has to be preserved to keep track of the `total` partial result while the rest of the recursion proceeds.

The key recognition point for this refactoring strategy is that we could remove our dependence on the stack by doing the addition *now* instead of *after*, and then forward-passing that partial result as an argument to the recursive call. In other words, instead of keeping `total` in the current function's stack frame, push it into the stack frame of the next recursive call; that frees up the current stack frame to be removed/reused.

To start, we could alter the signature our `sum(..)` function to have a new first parameter as the partial result:

```js
function sum(result,num1,...nums) {
	// ..
}
```

Now, we should pre-calculate the addition of `result` and `num1`, and pass that along:

```js
"use strict";

function sum(result,num1,...nums) {
	result = result + num1;
	if (nums.length == 0) return result;
	return sum( result, ...nums );
}
```

Now our `sum(..)` is in PTC form! Yay!

But the downside is we now have altered the signature of the function that makes using it stranger. The caller essentially has to pass `0` as the first argument ahead of the rest of the numbers they want to sum.

```js
sum( /*initialResult=*/0, 3, 1, 17, 94, 8 );		// 123
```

That's unfortunate.

Typically, people will solve this by naming their awkward-signature recursive function differently, then defining an interface function that hides the awkwardness:

```js
"use strict";

function sumRec(result,num1,...nums) {
	result = result + num1;
	if (nums.length == 0) return result;
	return sumRec( result, ...nums );
}

function sum(...nums) {
	return sumRec( /*initialResult=*/0, ...nums );
}

sum( 3, 1, 17, 94, 8 );								// 123
```

That's better. Still unfortunate that we've now created multiple functions instead of just one. Sometimes you'll see developers "hide" the recursive function as an inner function, like this:

```js
"use strict";

function sum(...nums) {
	return sumRec( /*initialResult=*/0, ...nums );

	function sumRec(result,num1,...nums) {
		result = result + num1;
		if (nums.length == 0) return result;
		return sumRec( result, ...nums );
	}
}

sum( 3, 1, 17, 94, 8 );								// 123
```

The downside here is that we'll recreate that inner `sumRec(..)` function each time the outer `sum(..)` is called. So, we can go back to them being side-by-side functions, but hide them both inside an IIFE, and expose just the one we want to:

```js
"use strict";

var sum = (function IIFE(){

	return function sum(...nums) {
		return sumRec( /*initialResult=*/0, ...nums );
	}

	function sumRec(result,num1,...nums) {
		result = result + num1;
		if (nums.length == 0) return result;
		return sumRec( result, ...nums );
	}

})();

sum( 3, 1, 17, 94, 8 );								// 123
```

OK, we've got PTC and we've got a nice clean signature for our `sum(..)` that doesn't require the caller to know about our implementation details. Yay!

But... wow, our simple recursive function has a lot more noise now. The readability has definitely been reduced. That's unfortunate to say the least. Sometimes, that's just the best we can do.

Luckily, in some other cases, like the present one, there's a better way. Let's reset back to this version:

```js
"use strict";

function sum(result,num1,...nums) {
	result = result + num1;
	if (nums.length == 0) return result;
	return sum( result, ...nums );
}

sum( /*initialResult=*/0, 3, 1, 17, 94, 8 );		// 123
```

What you might observe is that `result` is a number just like `num1`, which means that we can always treat the first number in our list as our running total; that includes even the first call. All we need is to rename those params to make this clear:

```js
"use strict";

function sum(num1,num2,...nums) {
	num1 = num1 + num2;
	if (nums.length == 0) return num1;
	return sum( num1, ...nums );
}

sum( 3, 1, 17, 94, 8 );								// 123
```

Awesome. That's much better, huh!? I think this pattern achieves a good balance between declarative/reasonable and performant.

Let's try refactoring with PTC once more, revisiting our earlier `maxEven(..)` (currently not PTC). We'll observe that similar to keeping the sum as the first argument, we can narrow the list of numbers one at a time, keeping the first argument as the highest even we've come across thus far.

For clarity, the algorithm strategy (similar to what we discussed earlier) we might use:

1. Start by comparing the first two numbers, `num1` and `num2`.
2. Is `num1` even, and is `num1` greater than `num2`? If so, keep `num1`.
3. If `num2` is even, keep it (store in `num1`).
4. Otherwise, fall back to `undefined` (store in `num1`).
5. If there are more `nums` to consider, recursively compare them to `num1`.
6. Finally, just return whatever value is left in `num1`.

Our code can follow these steps almost exactly:

```js
"use strict";

function maxEven(num1,num2,...nums) {
	num1 =
		(num1 % 2 == 0 && !(maxEven( num2 ) > num1)) ?
			num1 :
			(num2 % 2 == 0 ? num2 : undefined);

	return nums.length == 0 ?
		num1 :
		maxEven( num1, ...nums )
}
```

**Note:** The first `maxEven(..)` call is not in PTC position, but since it only passes in `num2`, it only recurses just that one level then returns right back out; this is only a trick to avoid repeating the `%` logic. As such, this call won't increase the growth of the recursive stack, any more than if that call was to an entirely different function. The second `maxEven(..)` call is the legitimate recursive call, and it is in fact in PTC position, meaning our stack won't grow as the recursion proceeds.

It should be repeated that this example is only to illustrate the approach to moving recursion to the PTC form to optimize the stack (memory) usage. The more direct way to express a max-even algorithm might indeed be a filtering of the `nums` list for evens first, followed then by a max bubbling or even a sort.

Refactoring recursion into PTC is admittedly a little intrusive on the simple declarative form, but it still gets the job done reasonably. Unfortunately, some kinds of recursion won't work well even with an interface function, so we'll need different strategies.

### Continuation Passing Style (CPS)

In JavaScript, the word *continuation* is often used to mean a function callback that specifies the next step(s) to execute after a certain function finishes its work. Organizing code so that each function receives another function to execute at its end, is referred to as Continuation Passing Style (CPS).

Some forms of recursion cannot practically be refactored to pure PTC, especially multiple recursion. Recall the `fib(..)` function earlier, and even the mutual recursion form we derived. In both cases, there are multiple recursive calls, which effectively defeats PTC memory optimizations.

However, you can perform the first recursive call, and wrap the subsequent recursive calls in a continuation function to pass into that first call. Even though this would mean ultimately many more functions will need to be executed in the stack, as long all of them, continuations included, are in PTC form, stack memory usage will not grow unbounded.

We could do this for `fib(..)`:

```js
"use strict";

function fib(n,cont = identity) {
	if (n <= 1) return cont( n );
	return fib(
		n - 2,
		n2 => fib(
			n - 1,
			n1 => cont( n2 + n1 )
		)
	);
}
```

Pay close attention to what's happening here. First, we default the `cont(..)` continuation function as our `identity(..)` utility from Chapter 3; remember, it simply returns whatever is passed to it.

Morever, not just one but two continuation functions are added to the mix. The first one receives the `n2` argument, which eventually receives the computation of the `fib(n-2)` value. The next inner continuation receives the `n1` argument, which eventually is the `fib(n-1)` value. Once both `n2` and `n1` values are known, they can be added together (`n2 + n1`), and that value is passed along to the next `cont(..)` continuation step.

Perhaps this will help mentally sort out what's going on: just like in the previous discussion when we passed partial results along instead of returning them back after the recursive stack, we're doing the same here, but each step gets wrapped in a continuation, which defers its computation. That trick allows us to perform multiple steps where each is in PTC form.

In static languages, CPS is often an opportunity for tail calls the compiler can automatically identify and rearrange recursive code to take advantage of. Unfortunately, that doesn't really apply to the nature of JS.

In JavaScript, you'd likely need to write the CPS form yourself. It's clunkier, for sure; the declarative notation-like form has certainly been obscured. But overall, this form is still more declarative than the `for`-loop imperative implementation.

**Warning:** One major caveat that should be noted is that in CPS, creating the extra inner continuation functions still consumes memory, but of a different sort. Instead of piling up stack frames, the closures just consume free memory (typically, from the heap). Engines don't seem to apply the `RangeError` limits in these cases, but that doesn't mean your memory usage is fixed in scale.

### Trampolines

Where CPS creates continuations and passes them along, another technique for alleviating memory pressure is called trampolines. In this style of code, CPS-like continuations are created, but instead of passed in, they are shallowly returned.

Instead of functions calling functions, the stack never goes beyond depth of one, because each function just returns the next function that should be called. A loop simply keeps running each returned function until there are no more functions to run.

One advantage with trampolines is you aren't limited to environments that support PTC; another is that each function call is regular, not PTC optimized, so it may run quicker.

Let's sketch out a `trampoline(..)` utility:

```js
function trampoline(fn) {
	return function trampolined(...args) {
		var result = fn( ...args );

		while (typeof result == "function") {
			result = result();
		}

		return result;
	};
}
```

As long as a function is returned, the loop keeps going, executing that function and capturing its return, then checking its type. Once a non-function comes back, the trampoline assumes the function calling is complete, and just gives back the value.

Because each continuation needs to return another continuation, we likely we'll need to use the earlier trick of forward-passing the partial result as an argument. Here's how we could use this utility with our earlier example of summation of a list of numbers:

```js
var sum = trampoline(
	function sum(num1,num2,...nums) {
		num1 = num1 + num2;
		if (nums.length == 0) return num1;
		return () => sum( num1, ...nums );
	}
);

var xs = [];
for (let i=0; i<20000; i++) {
	xs.push( i );
}

sum( ...xs );					// 199990000
```

The downside is that a trampoline requires you to wrap your recursive function in the trampoline driving function; moreover, just like CPS, closures are created for each continuation. However, unlike CPS, each continuation function returned runs and finishes right away, so the engine won't have to accumulate a growing amount of closure memory while the call stack depth of the problem is exhausted.

Beyond execution and memory performance, the advantage of trampolines over CPS is that they're less intrusive on the declarative recursion form, in that you don't have to change the function signature to receive a continuation function argument. Trampolines are not ideal, but they can be effective in your balancing act between imperative looping code and declarative recursion.

## Summary

Recursion is when a function recursively calls itself. Heh. A recursive definition for recursion. Get it!?

Direct recursion is a function that makes at least one call to itself, and it keeps dispatching to itself until it satisifies a base condition. Multiple recursion (like binary recursion) is when a function calls itself multiple times. Mutual recursion is when a two or more functions recursively loop by *mutually* calling each other.

The upside of recursion is that it's more declarative and thus typically more readable. The downside is usually performance, but more memory constraints even than execution speed.

Tail calls alleviate the memory pressure by reusing/discarding stack frames. JavaScript requires strict mode and proper tail calls (PTC) to take advantage of this "optimization". There are several techniques we can mix-n-match to refactor a non-PTC recursive function to PTC form, or at least avoid the memory constraints by flattening the stack.

Remember: recursion should be used to make code more readable. If you misuse or abuse recursion, the readability will end up worse than the imperative form. Don't do that!
