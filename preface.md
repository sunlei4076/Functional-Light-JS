# Functional-Light JavaScript
# 轻量函数式 JavaScript
# Preface
# 前言

> A monad is just a monoid in the category of endofunctors.

> 在范畴学中一个monad就是一个monoid。

Did I just lose you? Don't worry, I'd be lost, too! All those terms that only mean something to the already-initiated in Functional Programming&trade; (FP) are just jumbled nonsense to many of the rest of us.

有晕头转向吗？不要担心，我自己也被绕晕了！对于那些已经了解函数式编程的人来说，这些专业术语才有意义，然而对于大部分人而言，它们没有任何意义。

This book is not going to teach you what those words mean. If that's what you're looking for, keep looking. In fact, there's already plenty of great books that teach FP the *right way*, from the top-down. Those words have important meanings and if you formally study FP in-depth, you'll absolutely want to get familiar with them.

这本书并不打算教你以上那些专业术语的具体含义。如果那正是你想查找的，请继续查找。事实上，已经有很多从头到尾（正确的方式）介绍FP的书了。这些专业术语有很重要的意义，如果你在深入学习FP，你肯定会对这些专业术语越来越熟悉。

But this book is going to approach the topic quite differently. I'm going to present fundamental FP concepts from the ground-up, with fewer special or non-intuitive terms than most approaches to FP. We'll try to take a practical approach to each principle rather than a purely academic angle. **There will be terms**, no doubt. But we'll be careful and deliberate about introducing them and explaining why they're important.

但是本书打算以另一种方式讲解FP。我将从FP的一些基础概念讲起，并尽可能少用晦涩难懂的专业术语。我们将尝试以更实用的方法来探讨FP，而非纯粹的学术角度。毫无疑问，**肯定会有专业术语**。但是我将会小心谨慎的引入这些术语并解释为何它们如此重要。

Sadly, I am not a card-carrying member of the FP cool kids club. I've never been formally taught anything about FP. And though I have a CS academic background and I was decent at math, mathematical notation is not how my brain understands programming. I have never written a line of Scheme, Clojure, or Haskell. I'm not an old-school Lisp'r.

可悲的是我并非酷酷的FP儿童俱乐部的一员。我从没有正式学过FP。尽管我有计算机方面的教育背景并对数学有一定了解，但数学符号跟我理解的编程完全是两回事。我从来没写过一行Scheme，Clojure或Haskell代码。我也不是老派的Lisp‘r。

I *have* attended countless conference talks about FP, each one with the desperate clinging hope that finally, *this time*, would be the time I understood what this whole functional programming mysticism is all about. And each time, I came away frustrated and reminded that those terms got all mixed up in my head and I had no idea if or what I learned. Maybe I learned things. But I couldn't figure out what those things were, for the longest time.

我曾参加过不计其数的讨论FP的会议，每次都希望能彻底搞明白函数式编程中那些神秘的概念到底是什么意思。然而每次我都失望而归，那些概念在我脑海里乱成一团，我甚至不清楚自己学了些什么。也许我学到了些东西吧，但是很长时间我都不能确定自己学到了什么。

Little by little, across those various exposures, I teased out bits and pieces of important concepts that seem to just come all too naturally to the formal FPer. I learned them slowly and I learned them pragmatically and experientially, not academically with appropriate terminology. Have you ever known a thing for a long time, and only later found out it had a specific name you never knew!?

通过不断的编程实践，而非站在学术的角度，我慢慢的理解了那些对FPer来说很简单直白的重要概念。你是否也有类似的经历--你早就知道一件事，但直到很久之后你突然发现它竟然还有一个你从来不知道的名字！？

Maybe you're like me; I heard terms like "map-reduce" around industry segments like "big data" for years with no real idea what they were. Eventually I learned what the `map(..)` function did -- all long before I had any idea that list operations were a cornerstone of the FPer path and what makes them so important. I knew what *map* was long before I ever knew it was called `map(..)`.

也许你像我一样；好几年前就听说过像“map-reduce”，“big data”等这些术语，但并不懂它们的实际意义。最终我明白了`map(..)`函数到底做了哪些事情--在我知道列表操作是通向FPer之路的基石，并且为何它们如此重要之后。我知道*map*很久了，甚至在我知道它叫`map(..)`之前。

Eventually I began to gather all these tidbits of understanding into what I now call "Functional-Light Programming" (FLP).

最终我开始整理这些想法并将它们称之为“轻函数式编程”（FLP）

## Mission
## 使命

But, why is it so important for you to learn functional programming, even the light form?

但是，为什么学习函数式编程如此重要，即便只是学习轻函数式编程？

I've come to believe something very deeply in recent years, so much so you could *almost* call it a religious belief. I believe that programming is fundamentally about humans, not about code. I believe that code is first and foremost a means of human communication, and only as a *side effect* (hear my self-referential chuckle) does it instruct the computer.

最近几年我越来越深刻的理解到编程的核心是人，而不是代码，我甚至将其视为一种信仰。我坚信代码只是人类交流的手段，只有*副作用*（听到了自我引用的笑声）才对电脑发出具体指令。


The way I see it, functional programming is at its heart about using patterns in your code that are well-known, understandable, *and* proven to keep away the mistakes that make code harder to understand. In that view, FP -- or, ahem, FLP! -- might be one of the most important collections of tools any developer could acquire.
在我看来，函数式编程的核心在于让你在编程时使用一些广为人知、易于理解的模式。经过验证，这些模式可以有效隔离让代码难以理解的错误。所以，FP--FLP--是每个开发者都可以掌握的重要工具之一。

> The curse of the monad is that... once you understand... you lose the ability to explain it to anyone else.
>
> Douglas Crockford 2012 "Monads and Gonads"
>
> https://www.youtube.com/watch?v=dkZFtimgAcM

> monad的含义是指。。。一旦你搞懂了。。。你就无法跟别人解释什么是monad了。
>
> Douglas Crockford 2012 "Monads and Gonads"
>
> https://www.youtube.com/watch?v=dkZFtimgAcM

I hope this book "Maybe" breaks the spirit of that curse, even though we won't talk about "monads" until the very end in the appendices.

我希望这本书有可能打破上面的诅咒，尽管我们要到最后的附言部分才开始讨论“monads”


The formal FPer will often assert that the *real value* of FP is in using it essentially 100%: it's an all-or-nothing proposition. The belief is that if you use FP in one part of your program but not in another, the whole program is polluted by the non-FP stuff and therefore suffers enough that the FP was probably not worth it.

科班出身的FPer经常宣称只有100%使用FP才算是真正的使用FP：这是一种要么全有要么全无的主张。它会让人觉得如果编程时只在一部分使用了FP而另一部分没用到，整个程序会被那些没有使用FP的部分污染，从而认为使用FP并不值得。


I'll say unequivocally: **I think that absolutism is bogus**. That's as silly to me as suggesting that this book is only good if I use perfect grammar and active voice throughout; if I make any mistakes, it degrades the entire book's quality. Nonsense.

我想明确的说：**我认为绝对主义并不存在**。这和愚蠢的建议我只有使用完美的语法这本书才算完美，如果犯了一点点错误都会让整本书降级一样。没有意义。

The better I am at writing in a clear, consistent voice, the better this book experience will be for you. But I'm not a 100% perfect author. Some parts will be better written than others. The parts where I can still improve are not going to invalidate the other parts of this book which are useful.

我写的越清楚，前后越一致，你阅读此书的体验将越来越好。但我不是一个完美无缺的作者。有些章节可能比另外一些写的好。那些有待提高的章节不会使书中写的好的部分黯然失色。

And so it goes with our code. The more you can apply these principles to more parts of your code, the better your code will be. Use them well 25% of the time, and you'll get some good benefit. Use them 80% of the time, and you'll see even more benefit.

同样的道理也适用于代码。随着你越来越多的使用FP中的模式，你的代码质量会越来越高。25%的时间使用它们，你会得到一些好处。80%的时间使用它们，你将收益更多。

With perhaps a few exceptions, I don't think you'll find many absolutes in this text. We'll instead talk about aspirations, goals, principles to strive for. We'll talk about balance and pragmatism and trade-offs.

除了几处仅存的特例，你不会在本书里看到很多绝对式的论断。相反，我们讨论的是要追求的目标和现实中方方面面的权衡。

Welcome to this journey into the most useful and practical foundations of FP. We both have plenty to learn!

欢迎来到最实用的FP学习之旅。我们将共同探讨学习！
