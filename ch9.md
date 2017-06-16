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

我们来谈谈递归吧。在我们入坑之前，请查阅上一页的正式定义。

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

递归深谙函数式编程之精髓，最被广泛引证的原因是，在调用栈中，递归把(大部分)显式状态跟踪换为了隐式状态。通常，当问题需要条件分支和回溯计算，这时候递归非常有用，并且在纯迭代环境中管理这种状态，是相当棘手的；最起码，这些代码是不可或缺且晦涩难懂。但是在堆栈上调用每一级的分支作为其自己的作用域，很明显，这通常会影响到代码的可读性。

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

每个函数调用都将开辟出一小块称为堆栈帧的内存。堆栈帧中包含了函数语句当前状态的某些重要信息，包括任意变量的值。之所以这样，是因为一个函数暂停去执行另外一个函数，而另外一个函数运行结束后，引擎需要返回到之前暂停时候的状态继续执行。

当第二个函数开始执行，堆栈帧增加到 2 个。如果第二个函数又调用了另外一个函数，堆栈帧将增加到 3 个，以此类推。“堆、栈”的意思是，函数被它前一个函数调用时，这个函数帧会被“推”到最顶部。当这个函数调用结束后，它的帧会从堆栈中退出。

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

来一步步想象下这个程序的堆栈帧：

<p align="center">
	<img src="fig15.png" width="600">
</p>

**注意：** 如果这些函数间没有相互调用，而只是依次执行 -- 比如前一个函数运行结束后才开始调用下一个函数 `baz(); bar(); foo();` -- 则堆栈帧并没有产生；因为在下一个函数开始之前，上一个函数运行结束并把它的帧从堆栈里面移除了。

所以，每一个函数运行时候，都会占用一些内存。对多数程序来说，这没什么大不了的，不是吗？但是，一旦你引用了递归，就有什么大不了咯。
虽然你几乎肯定不会在一个调用栈中手动调用成千（或数百）次不同的函数，但你很容易看到产生数万个或更多递归调用的堆栈。

当引擎认为调用栈增加的太多并且应该停止增加时候，它会以任意的限制来阻止当前步骤，所以 `isOdd(..)` 或 `isEven(..)` 函数抛出了 `RangeError` 未知错误。这不太可能是内存接近零时候产生的限制，而是引擎的预测，因为如果这种程序持续运行下去，内存会爆掉的。由于引擎无法判断一个程序最终是否会停止，所以它必须做出确定的猜测。

引擎的限制因情况而定。规范里面并没有任何说明，因此，它也不是 *必需的*。但如果没有限制的话，设备很容易遭到破坏或恶意代码攻击，故而几乎所有的JS引擎都有一个限制。不同的设备环境、不同的引擎，会有不同的限制，也就无法预测或保证函数调用栈能调用多少次。

在处理大数据量时候，这个限制对于开发人员来说，会对递归的性能有一定的要求。我认为，这种限制也可能是造成开发人员不喜欢使用递归编程的最大原因。
遗憾的是，递归编程是一种编程思想而不是主流的编程技术。

### 尾调用

递归编程和内存限制都要比 JS 技术出现的早。追溯到上世纪 60 年代，当时开发人员想使用递归编程并希望运行在他们强大的计算机的设备，而所谓强大计算机的内存，尚远不如我们今天在手表上的内存。

幸运的是，在那个希望的原野上，进行了一个有力的观测。该技术称为 **尾调用**。

它的思路是如果一个回调从函数 `baz()` 转到函数 `bar()` 时候，而回调是在函数 `baz()` 的最底部执行 -- 也就是尾调用 -- 那么 `baz()` 的堆栈帧就不再需要了。也就意谓着，内存可以被回收，或只需简单的执行 `bar()` 函数。 如图所示：

<p align="center">
	<img src="fig16.png" width="600">
</p>

尾调用并不是递归特有的；它适用于任何函数调用。但是，在大多数情况下，你的手动非递归调用栈不太可能超过 10 级，因此尾调用对你程序内存的影响可能相当低。

在递归的情况下，尾调用作用很明显，因为这意味着递归堆栈可以“永远”运行下去，唯一的性能问题就是计算，而不再是固定的内存限制。在固定的内存中尾递归可以运行 `O(1)` （常数阶时间复杂度计算）。

这些技术通常被称为尾调用优化（TCO），但重点在于从优化技术中，区分出在固定内存空间中检测尾调用运行的能力。从技术上讲，尾调用并不像大多数人所想的那样，它们的运行速度可能比普通回调还慢。TCO 是关于把尾调用更加高效运行的一些优化技术。

### 正确的尾调用 (PTC)

在 ES6 出来之前，JavaScript 对尾调用一直没明确规定（也没有禁用）。ES6 明确规定了 PTC 的特定形式，在 ES6 中，只要使用尾调用，就不会发生栈溢出。实际上这也就意谓着，只要正确的使用 PTC，就不会抛出 `RangeError` 这样的异常错误。

首先，在 JavaScript 中应用 PTC，必须以严格模式书写代码。如果你以前没有用过严格模式，你得试着用用了。那么，您，应该已经在使用严格模式了吧！？

其次，*正确* 的尾调用就像这个样子：

```js
return foo( .. );
```

换句话说，函数调用应该放在最后一步去执行，并且不管返回什么东东，都得有返回（ `return` ）。这样的话，JS 就不再需要当前的堆栈帧了。

下面这些 *不能* 称之为 PTC：

```js
foo();
return;

// 或

var x = foo( .. );
return x;

// 或

return 1 + foo( .. );
```

**注意：** 一些 JS 引擎 *能够* 把 `var x = foo(); return x;` 自动识别为 `return foo();`，这样也可以达到 PTC 的效果。但这毕竟不符合规范。

`foo(..)` 运行结束之后 `1+` 这部分才开始执行，所以此时的堆栈帧依然存在。

不过，下面这个 *是* PTC：

```js
return x ? foo( .. ) : bar( .. );
```

`x` 进行条件判断之后，或执行 `foo(..)`，或执行 `bar(..)`，不论执行哪个，返回结果都会被 `return` 返回掉。这个例子符合 PTC 规范。

为了避免堆栈增加，PTC 要求所有的递归必须是在尾部调用，因此，二分法递归 -- 两次（或以上）递归调用 -- 是不能实现 PTC 的。我们曾在文章的前面部分展示过把二分法递归转变为相互递归的例子。也许我们可以试着化整为零，把多重递归拆分成符合 PTC 规范的单个函数回调。

## 重构递归

如果你想用递归来处理问题，却又超出了 JS 引擎的内存堆栈，这时候就需要重构下你的递归调用，使它能够符合 PTC 规范（或着避免嵌套调用）。这里有一些重构方法也许可以用到，但需要根据实际情况权衡。

可读性强的代码，是我们的终级目标 -- 谨记，谨记。如果使用递归后会造成代码难以阅读/理解，那就 **不要使用递归**；换个容易理解的方法吧。

### 更换堆栈

对递归来说，最主要的问题是它的内存使用情况。保持堆栈帧跟踪函数调用的状态，并将其分派给下一个递归调用迭。如果我们弄清楚了如何重新排列我们的递归，就可以用 PTC 实现递归，并利用 JS 引擎对尾调用的优化处理，那么我们就不用在内存中保留当前的堆栈帧了。

来回顾下之前用到的一个求和的例子：

```js
function sum(num1,...nums) {
	if (nums.length == 0) return num1;
	return num1 + sum( ...nums );
}
```

这个例子并不符合 PTC 规范。`sum(...nums)` 运行结束之后，`num1` 与 `sum(...nums)` 的运行结果进行了累加。这样的话，当其余参数 `...nums` 再次进行递归调用时候，为了得到其与 `num1` 累加的结果，必须要保留上一次递归调用的堆栈帧。

重构策略的关键点在于，我们可以通过把 `置后` 处理累加改为 `提前` 处理，来消除对堆栈的依赖，然后将该部分结果作为参数传递到递归调用。换句话说，我们不用在当前运用函数的堆栈帧中保留 `num1 + sum(...num1)` 的总和，而是把它传递到下一个递归的堆栈帧中，这样就能释放当前递归的堆栈帧。

开始之前，我们做些改动：把部分结果作为一个新的第一个参数传入到函数 `sum(..)`：

```js
function sum(result,num1,...nums) {
	// ..
}
```

这次我们先把 `result` 和 `num1` 提前计算，然后把 `result` 作为参数一起传入：

```js
"use strict";

function sum(result,num1,...nums) {
	result = result + num1;
	if (nums.length == 0) return result;
	return sum( result, ...nums );
}
```

现在 `sum(..)` 已经符合 PTC 优化规范了！Yes！

但是还有一个缺点，我们修改了函数的参数传递形式后，用法就跟以前不一样了。调用者不得不在需要求和的那些参数的前面，再传递一个 `0` 作为第一个参数。

```js
sum( /*initialResult=*/0, 3, 1, 17, 94, 8 );		// 123
```

这就尴尬了。

通常，大家的处理方式是，把这个尴尬的递归函数重新命名，然后定义一个接口函数把问题隐藏起来：

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

情况好了些。但依然有问题：之前只需要一个函数就能解决的事，现在我们用了两个。有时候你会发现，在处理这类问题上，有些开发者用内部函数把递归 “藏了起来”：

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

这个方法的缺点是，每次调用外部函数 `sum(..)`，我们都得重新创建内部函数 `sumRec(..)`。我们可以把他们平级放置在立即执行的函数中，只暴露出我们想要的那个的函数：

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

好啦，现在即符合了 PTC 规范，又保证了 `sum(..)` 参数的整洁性，调用者不需要了解函数的内部实现细节。完美！

可是...天呐，本来是简单的递归函数，现在却出现了很多噪点。可读性已经明显降低。至少说，这是不成功的。有些时候，这只是我们能做的最好的。

幸运的事，在一些其它的例子中，比如上一个例子，有一个比较好的方式。一起重新看下：

```js
"use strict";

function sum(result,num1,...nums) {
	result = result + num1;
	if (nums.length == 0) return result;
	return sum( result, ...nums );
}

sum( /*initialResult=*/0, 3, 1, 17, 94, 8 );		// 123
```

也许你会注意到，`result` 就像 `num1` 一样，也就是说，我们可以把列表中的第一个数字作为我们的运行总和；这甚至包括了第一次调用的情况。我们需要的是重新命名这些参数，使函数清晰可读：

```js
"use strict";

function sum(num1,num2,...nums) {
	num1 = num1 + num2;
	if (nums.length == 0) return num1;
	return sum( num1, ...nums );
}

sum( 3, 1, 17, 94, 8 );								// 123
```

帅呆了。比之前好了很多，嗯？！我认为这种模式在声明/合理和执行之间达到了很好的平衡。

让我们试着重构下前面的 `maxEven(..)`（目前还不符合 PTC 规范）。就像之前我们把参数的和作为第一个参数一样，我们可以依次减少列表中的数字，同时一直把遇到的最大偶数作为第一个参数。

为了清楚起见，我们可能使用算法策略（类似于我们之前讨论过的）：

1. 首先对前两个参数 `num1` 和 `num2` 进行对比。
2. 如果 `num1` 是偶数，并且 `num1` 大于 `num2`，`num1` 保持不变。
3. 如果 `num2` 是偶数，把 `num2` 赋值给 `num1`。
4. 否则的话，`num1` 等于 `undefined`。
5. 如果除了这两个参数之外，还存在其它参数 `nums`，把它们与 `num1` 进行递归对比。
6. 最后，不管是什么值，只需返回 `num1`。

依照上面的步骤，代码如下：

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

**注意：** 函数第一次调用 `maxEven(..)` 并不是为了 PTC 优化，当它只传递 `num2` 时，只递归一级就返回了；它只是一个避免重复 `％` 逻辑的技巧。因此，只要该调用是完全不同的函数，就不会增加递归堆栈。第二次调用 `maxEven(..)` 是基于 PTC 优化角度的真正递归调用，因此不会随着递归的进行而造成堆栈的增加。

重申下，此示例仅用于说明将递归转化为符合 PTC 规范以优化堆栈（内存）使用的方法。求最大偶数值的更直接方法可能是，先对参数列表中的 `nums` 过滤，然后冒泡或排序处理。

基于 PTC 重构递归，固然对简单的声明形式有一些影响，但依然有理由去做这样的事。不幸的是，存在一些递归，即使我们使用了接口函数来扩展，也不会很好，因此，我们需要有不同的思路。

### Continuation Passing Style (CPS)

在 JavaScript 中， *continuation* 一词通常用于表示一个回调函数，
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
