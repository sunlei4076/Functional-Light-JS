# 轻量级函数式编程
# 第一章：为什么使用函数式编程？

> Functional programmer: (noun) One who names variables "x", names functions "f", and names code patterns "zygohistomorphic prepromorphism"
>
> James Iry ‏@jamesiry 5/13/15
>
> https://twitter.com/jamesiry/status/598547781515485184

函数式编程（以下简称FP）不是一个新的概念，这个概念几乎贯穿了整个编程史。我不确定这么说是否合理，但是很确定的一点是：直到最近几年，FD才成为整个开发界的主流观念。所以我觉得FD更像学者的领域。

然而一切都在变。无论是语言，还是库和框架，都对FP的兴趣空前高涨。你也很可能在读相关内容，因为你终于意识到FP是不容忽视的东西。或者你跟我一样，已经很多次尝试着去学FP，但却很难理解所有的术语或数学符号。

无论你出于何目的翻阅本书，欢迎加入我们！

## 置信度

我有一个非常简单的前提，这是我作为软件开发老师（JavaScript）中所做的一切的基础：你不能信任的代码是你不明白的代码。此外，对你不信任或不明白的代码，那么你就会对这样的代码不自信，并且会认为它不适合开发你的业务。当代码运行的时候也只能祈求好运。

信任是什么意思？信任代表你通过读代码而不只是运行，你能证实你理解这段代码能干什么事，而不只是停留在它可能是干什么。也许我们不应该总倾向于通过运行测试程序，从而来验证我们的程序的正确性。我并不是说测试不好，而是说我们应该期待去能够很好地了解代码，这样我们在运行测试代码之前就会知道它肯定能跑通。

通过读代码能对我们的程序有更多信心，我相信构成FP基础的技术，是本着心态设计的。 理解FP并在程序中用心实践的人，通过FP已经被证实的原则，能够写出他们可读性高和可验证的代码，来达到他们想要的目的。

我希望您能通过理解轻量级函数式编程的原则，对您编写的代码更有信心，并且能在之后的路上越走越好。

## 交流渠道

函数式编程为何重要？为了回答这个问题，我们先退一万步，来讨论一下编程本身的重要性。

我认为代码不是构成电脑的说明，这么说你可能感到很奇怪。事实上，代码能指示电脑如何运行就是一个意外中的惊喜。

作为与他人交流的一种手段，我深信代码的作用非常重要。

根据以往经验你可能知道，有时候花很多时间写代码其实是为了搞明白现有的代码。我们的大部分时间其实都是在维护别人的代码（或自己的老代码），只有少部分时间是在敲新代码。

你知道研究过这个话题的研究人员说给出了怎样的数据吗？70％ 的时间我们花费维护代码只是花在阅读并理解代码。 也难怪全球程序员的平均代码行数是5行。 我们一天花了7个半小时才读懂代码，然后找出哪5行代码奏效。

我想我们应该更多的关注一下代码的可读性。可能的话，不妨多花点时间在可读性上。顺便提一句，可读性并不只是说代码量少，对代码的熟悉程度会影响代码的可读性（这一点也是被证实过的）。

因此，如果我们要花费更多的时间来关注编写代码的可读性和可理解性，所以FP就是这样一个非常方便的模式。FP的原则是完善的，经过了深入的研究和审查，并且可被证明和验证。

如果我们使用FP原则，我相信我们将创建更容易理解的代码。一旦我们知道这些原则，它们将在代码中被识别和熟悉，这意味着当我们读取一段代码时，我们将花费更少的时间来进行定位。我们的重点将在于如何组建所有已知的乐高片段，而不是这些乐高片段意味着什么。

FP是编写可读代码的最有效工具之一(可能还有其他)。这就是为什么FP如此重要。

### 可读性曲线

很重要的是，我花了一年时间来讲述一种多年来让我感到困惑和沮丧的现象，在写这本书时特别尖锐。

这也可能是许多开发人员倾向于碰到的东西。亲爱的读者，当你读这篇文章的时候，你可能会发现自己也在这条船上。但是振作起来，如果坚持下去，陡峭的学习曲线总会过去。

<p align="center">
	<img src="fig17.png" width="600">
</p>

我们将在下一章更深入的讨论这个问题。但是一些命令语句是已经用过的，像 if 语句和 for 循环这样的语句。这些语句旨在精确地指导计算机“如何”完成一件事情。声明式代码，以及我们将努力学习遵循FP原则编写的代码，是更专注于描述产出“什么”代码。

还有个残酷的问题摆在眼前，这个问题花费了我在这本书上的很多时间：我需要花费更多的精力和更多的代码来提高代码的可读性，尽量减少或消除你可能编写错误的大部分地方。

如果你希望用FP重构过的代码能够立刻变得更美观、优雅、智能和简洁的话，这个有点不太现实，这个变化是需要一个过程的。

FP的构思是很奇特的，代码应该如何基于它进行构造，使得数据流更加明显，并能让读者很快理解你的思想。这种努力是非常值得的，然而过程很艰辛，你可能需要花很多时间基于FP来调整代码直到代码可读性变得好一些。

另外，我的经验是，将命令式代码转换为更偏向于声明式的FP之前，大约需要做六次尝试。对我来说，编写FP的代码更像是一个过程，而不是从一个范例到另一个范例的二进制转换。

我也会经常对写过的代码进行再读。就是说，写完一段代码，过几个小时或一天再看就会有不一样的感觉。通常，重写的代码是比较混乱不堪，所以需要反复调整。

函数式编程并没有让我在艺术家的画布上实现一个优雅的笔触，让观众敬畏。相反，它是一个艰苦的，详细的，有时是阴险的通过一个被忽视的领域杂草的破解。 函数式编程的过程并没有让我在艺术的画布上笔下生辉，让观众拍岸叫好。相反，编程的过程很艰辛且历历在目，感觉像坐在一辆不知去向的马车穿过一片荒芜的灌木丛林。

我不是试图打压你的激情，真切希望你也能够一路披荆斩棘。过后我终于看到可读性曲线向上延伸，所有付出都是值得的，我相信你也将会有这样的感受。

## Take

我们要系统的学习FP，探索发现最基本的原则，我相信正规的FP用户会遵循这些原则并把它们作为开发的框架。但在大多数情况下，我们大都选择避开晦涩的术语或数学符号，否则很容易使学习者受挫。

我觉得一项技术你怎么称呼它不重要，重要的是理解它是什么并且它是怎么工作的。这并不是说共享术语不重要，它无疑可以简化经验丰富的专业人士之间的交流。但对学习者来说，它有点分散人的注意力。

所以我希望这本书能更多地关注基本概念而不是花哨的术语。这并不是说没有术语，肯定会有。但不要太沉迷于华丽的词藻，追寻其背后的含义，这正是本书的目的。

我把这种欠缺正轨实践的编程思想称为“轻量级函数编程”，因为我认为真正的FP的形式主义在于， 如果你还不习惯它主张的思想，你可能很难用它。这不仅仅只是猜测，而是我的亲身经历。即使在传教FP过程和完成这本书之后，我仍然可以说，FP中术语和符号的形式化对于我来说是非常非常困难的。我已经再三尝试，发现大部分都是很掌握的。

我知道很多FP用户会认为形式主义本身有助于学习。但我认为这是一个坑，只有当你用形式主义获得某种安慰时，你就会踩坑。如果你碰巧已经有了数学背景，甚至有一些 CS 经验，这对你来说可能驾轻就熟。但是我们中的一些人不具备这些条件，不管我们怎么努力，形式主义总是阻碍我们前进。

因此，这本书介绍了一些我认为FP会涉及到的概念，虽然不能直接让你受益但可以帮你逐步理解FP整个过程。 

## 你不需要它

如果你规划一个项目花了很长时间，那么别人一定会告诉你“YAGNI”：“你不需要它”。这个原则主要来自极限编程，强调构建特性的高风险和成本，这个风险和成本源自于项目本身是否需要。

Sometimes we guess we'll need a feature in the future, build it now believing it'll be easier to do as we build other stuff, then realize we guessed wrong and the feature wasn't needed, or needed to be quite different. Other times we guess right, but build a feature too early, and suck up time from the features that are genuinely needed now; we incur an opportunity cost in diluting our energy.

YAGNI challenges us to remember: even if it's counter intuitive in a situation, we often should postpone building something until it's presently needed. We tend to exaggerate our mental estimates of the future refactoring cost of adding it later when it is needed. Odds are, it won't be as hard to do later as we might assume.

As it applies to functional programming, I would give this admonition: there will be plenty of interesting and compelling patterns discussed in this text, but just because you find some pattern exciting to apply, it may not necessarily be appropriate to do so in a given part of your code.

This is where I will differ from many more formal FPers: just because you *can* FP something doesn't mean you *should* FP it. Moreover, there are many ways to slice a problem, and even though you may have learned a more sophisticated approach that is more "future-proof" to maintenance and extensibility, a simpler FP pattern might be more than sufficient in that spot.

Generally, I'd recommend to seek balance in what you code, and to be conservative in your application of FP concepts as you get the hang of things. Default to the YAGNI principle in deciding if a certain pattern or abstraction will help that part of the code be more readable or if it's just introducing more clever sophistication that isn't (yet) warranted.

> Reminder, any extensibility point that’s never used isn’t just wasted effort, it’s likely to also get in your way as well
>
> Jeremy D. Miller @jeremydmiller 2/20/15
>
> https://twitter.com/jeremydmiller/status/568797862441586688

Remember, every single line of code you write has a reader cost associated with it. That reader may be another team member, or even your future self. Neither of those readers will be impressed with overly clever unnecessary sophistication to show off your FP agility.

The best code is the code that is most readable in the future because it strikes exactly the right balance between what it can/should be (idealism) and what it must be (pragmatism).

## Resources

I have drawn on a great many different resources to be able to compose this text. I believe you too may benefit from them, so I wanted to take a moment to point them out.

### Books

Some FP/JavaScript books that you should definitely read:

* [Professor Frisby's Mostly Adequate Guide to Functional Programming](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch1.html) by [Brian Lonsdorf](https://twitter.com/drboolean)
* [JavaScript Allongé](https://leanpub.com/javascript-allonge) by [Reg Braithwaite](https://twitter.com/raganwald)
* [Functional JavaScript](http://shop.oreilly.com/product/0636920028857.do) by [Michael Fogus](https://twitter.com/fogus)

### Blogs/Sites

Some other authors and content you should check out:

* [Fun Fun Function Videos](https://www.youtube.com/watch?v=BMUiFMZr7vk) by [Mattias P Johansson](https://twitter.com/mpjme)
* [Awesome FP JS](https://github.com/stoeffel/awesome-fp-js)
* [Kris Jenkins](http://blog.jenkster.com/2015/12/what-is-functional-programming.html)
* [Eric Elliott](https://medium.com/@_ericelliott)
* [James A Forbes](https://james-forbes.com/)
* [James Longster](https://github.com/jlongster)
* [André Staltz](http://staltz.com/)
* [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon#functional-programming-jargon)
* [Functional Programming Exercises](https://github.com/InceptionCode/Functional-Programming-Exercises)

### Libraries

The code snippets in this book do not use libraries. Each operation that we discover, we'll derive how to implement it in standalone, plain ol' JavaScript. However, as you begin to build more of your real code with FP, you'll quickly want a library to provide optimized and highly reliable versions of these commonly accepted utilities.

By the way, you'll want to make sure you check the documentation for the library functions you use to make sure you know how they work. There will be a lot of similarities in many of them to the code we build on in this text, but there will undoubtedly be some differences, even between popular libraries.

Here are a few popular FP libraries for JavaScript that are a great place to start your exploration with:

* [Ramda](http://ramdajs.com)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)

Appendix C illustrates some of these libraries using various examples from the text.

## 总结

这就是轻量式的 JavaScript。我们的目标是学会与代码交流，而不是在符号或术语的大山下被压的喘不过气。希望这本书能开启你的旅程！
