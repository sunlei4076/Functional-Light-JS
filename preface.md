# Functional-Light JavaScript
# 轻函数JavaScript
# Preface
# 前言

> A monad is just a monoid in the category of endofunctors.
> 在范畴学中一个monad就是一个monoid。

Did I just lose you? Don't worry, I'd be lost, too! All those terms that only mean something to the already-initiated in Functional Programming&trade; (FP) are just jumbled nonsense to many of the rest of us.
我有让你晕头转向吗？不要担心，我自己也被绕晕了！对于那些已经了解函数式编程的人来说，这些专业术语才有意义，然而对于大部分人，这些专业术语没有任何意义。

This book is not going to teach you what those words mean. If that's what you're looking for, keep looking. In fact, there's already plenty of great books that teach FP the *right way*, from the top-down. Those words have important meanings and if you formally study FP in-depth, you'll absolutely want to get familiar with them.
这本书并不打算教你上面那些专业术语的具体含义。如果这正是你想查找的，请继续查找。事实上，已经有很多从上到下（正确的方式）介绍FP的书了。这些专业术语有很重要的意义，如果你在正式的深入学习FP，你肯定会对这些专业术语越来越熟悉。

But this book is going to approach the topic quite differently. I'm going to present fundamental FP concepts from the ground-up, with fewer special or non-intuitive terms than most approaches to FP. We'll try to take a practical approach to each principle rather than a purely academic angle. **There will be terms**, no doubt. But we'll be careful and deliberate about introducing them and explaining why they're important.
但是本书打算以另一种方式讲解FP。我将从FP的一些基础概念讲起，并尽可能少用晦涩难懂的专业术语。我们将尝试以更实用的方法来探讨FP，而非纯粹的学术角度。毫无疑问，**肯定会有专业术语**。但是我将会小心谨慎的引入这些术语并解释为何它们如此重要。

Sadly, I am not a card-carrying member of the FP cool kids club. I've never been formally taught anything about FP. And though I have a CS academic background and I was decent at math, mathematical notation is not how my brain understands programming. I have never written a line of Scheme, Clojure, or Haskell. I'm not an old-school Lisp'r.
可悲的是我并非酷酷的FP儿童俱乐部成员。我从没有正式学过FP。尽管我有CS的教育背景并对数学有一定了解，但数学符号跟我理解的编程完全是两回事。我从来没写过一行Scheme，Clojure或Haskell代码。我也不是老派的Lisp‘r。

I *have* attended countless conference talks about FP, each one with the desperate clinging hope that finally, *this time*, would be the time I understood what this whole functional programming mysticism is all about. And each time, I came away frustrated and reminded that those terms got all mixed up in my head and I had no idea if or what I learned. Maybe I learned things. But I couldn't figure out what those things were, for the longest time.
我曾参加过不计其数的讨论FP的会议，每次都希望能彻底搞明白函数式编程中那些神秘的概念到底是什么意思。然而每次我都失望而归，那些概念在我脑海里乱成一团，我甚至不清楚自己学了些什么。也许我学到了东西吧，但是很长时间我都不能确定自己学到了什么。

Little by little, across those various exposures, I teased out bits and pieces of important concepts that seem to just come all too naturally to the formal FPer. I learned them slowly and I learned them pragmatically and experientially, not academically with appropriate terminology. Have you ever known a thing for a long time, and only later found out it had a specific name you never knew!?
通过不断的编程实践，而非站在学术的角度，我慢慢的学到了那些对FPer来说很简单直白的重要概念。你之前是否知道一件事物，但直到很久之后你突然发现它竟然还有一个你从来不知道的名字！？

Maybe you're like me; I heard terms like "map-reduce" around industry segments like "big data" for years with no real idea what they were. Eventually I learned what the `map(..)` function did -- all long before I had any idea that list operations were a cornerstone of the FPer path and what makes them so important. I knew what *map* was long before I ever knew it was called `map(..)`.



Eventually I began to gather all these tidbits of understanding into what I now call "Functional-Light Programming" (FLP).

## Mission

But, why is it so important for you to learn functional programming, even the light form?

I've come to believe something very deeply in recent years, so much so you could *almost* call it a religious belief. I believe that programming is fundamentally about humans, not about code. I believe that code is first and foremost a means of human communication, and only as a *side effect* (hear my self-referential chuckle) does it instruct the computer.

The way I see it, functional programming is at its heart about using patterns in your code that are well-known, understandable, *and* proven to keep away the mistakes that make code harder to understand. In that view, FP -- or, ahem, FLP! -- might be one of the most important collections of tools any developer could acquire.

> The curse of the monad is that... once you understand... you lose the ability to explain it to anyone else.
>
> Douglas Crockford 2012 "Monads and Gonads"
>
> https://www.youtube.com/watch?v=dkZFtimgAcM

I hope this book "Maybe" breaks the spirit of that curse, even though we won't talk about "monads" until the very end in the appendices.

The formal FPer will often assert that the *real value* of FP is in using it essentially 100%: it's an all-or-nothing proposition. The belief is that if you use FP in one part of your program but not in another, the whole program is polluted by the non-FP stuff and therefore suffers enough that the FP was probably not worth it.

I'll say unequivocally: **I think that absolutism is bogus**. That's as silly to me as suggesting that this book is only good if I use perfect grammar and active voice throughout; if I make any mistakes, it degrades the entire book's quality. Nonsense.

The better I am at writing in a clear, consistent voice, the better this book experience will be for you. But I'm not a 100% perfect author. Some parts will be better written than others. The parts where I can still improve are not going to invalidate the other parts of this book which are useful.

And so it goes with our code. The more you can apply these principles to more parts of your code, the better your code will be. Use them well 25% of the time, and you'll get some good benefit. Use them 80% of the time, and you'll see even more benefit.

With perhaps a few exceptions, I don't think you'll find many absolutes in this text. We'll instead talk about aspirations, goals, principles to strive for. We'll talk about balance and pragmatism and trade-offs.

Welcome to this journey into the most useful and practical foundations of FP. We both have plenty to learn!
