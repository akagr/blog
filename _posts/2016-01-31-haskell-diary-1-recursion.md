---
toc: true
layout: post
description:
categories: []
title: Haskell Diary 1 - Recursion
---
Haskell is the first pure functional programming language that I have had a serious contact with. I'm very much a noob right now but I've found that there's a lot of gold to be found right from day 1 in functional world. So I've thought about documenting things which I found really cool, mind bending or which simply took a long time for me to wrap my head around (still, cool).

So... recursion.

I'll get killed in the street if I said that Haskell can do recursion. Of course Haskell can do recursion. C can do recursion. Javascript can do recursion. Ruby, Java (and most other languages) can do it too.

The reason why I'm talking about recursion in Haskell is because of its support for infinite lists. More specifically, the reason I'm *really* talking about recursion is because of an example I came across which blew me away. Some background for the uninitiated first.

> Recursion means a function calling itself

Bet anyone reading this already knew that. A classic example of recursion is fibonacci series. So here's a naive program which probably every programmer has seen in their language(s) of choice.

```haskell
fibonacci 0 = 0
fibonacci 1 = 1
fibonacci x = fibonacci (x - 1) + fibonacci (x - 2)
``` 

The reason it's called naive is because it's neither the most efficient nor the most elegant way of doing things. On my 2014 macbook pro with core i5, `fibonacci 1` gives result instantly. `fibonacci 25` seems a fraction of a second slower. `fibonacci 50` hasn't yielded results yet and I executed it 11 minutes ago.

Back on track, I came across following implementation of fibonacci while learning the basics of Haskell. It gives me results for 1000 instantaneously and doesn't involves memoization or other state-dependent techniques.

```haskell
fibs = 0 : 1 : zipWith (+) fibs (tail fibs)
```

Before diving in the down and low of it, following are (hopefully) self-explanatory examples of some other functions used here.

```haskell
-- tail takes a list and gives it back after removing its first element
tail [1, 2, 3, 4, 5] -- > [2, 3, 4, 5]
tail "haskell diary" -- > "askell diary" 

-- zipWith takes a function and two lists and return the
-- result of applying that function on corresponding elements
zipWith (+) [1, 2, 3] [4, 5, 6] -- > [5, 7, 9]
zipWith (*) [1, 2, 3] [2..] -- > [2, 6, 12]
```

Also, let's reduce some noise by replacing `zipWith (+)` by a function which does the same but would look more at-home here. Also, rewrite the above code with our substituted function.

```haskell
addLists = zipWith (+)
-- addList [1, 2, 3] [4, 5, 6] gives [5, 7, 9]

fibs = 0 : 1 : addLists fibs (tail fibs)
```

Makes better sense. Now, this code generates an infinitely long fibonacci sequence. It does that by recursively adding a list to itself only the second time it shifts it position (using tail) by a place. Let's try and break it down.

```haskell
fibs = 0 : 1 : addLists fibs (tail fibs)

-- expanding by substituting fibs with its own definition
fibs = 0 : 1 : addLists (0 : 1 : addLists fibs (tail fibs)) (1 : addLists fibs (tail fibs))

-- expanding again by replacing all instances of fibs by
-- its definition above
-- warning: keep your brains together now. I'm just replacing stuff.
fibs = 0 : 1 : addLists (0 : 1 : addLists (0 : 1 : addLists (0 : 1 : addLists fibs (tail fibs)) (1 : addLists fibs (tail fibs))) (1 : addLists (0 : 1 : addLists fibs (tail fibs)) (1 : addLists fibs (tail fibs))) (1 : addLists (0 : 1 : addLists (0 : 1 : addLists fibs (tail fibs)) (1 : addLists fibs (tail fibs))) (1 : addLists (0 : 1 : addLists fibs (tail fibs)) (1 : addLists fibs (tail fibs)))
```

And it will go on. Point of interest is that, after each expansion, we can apply `addLists` to get a number out. Let's not do any further expansion (and risk fainting) and instead start working our way back to simplify by discarding and condensing. So we take as much as is concrete (does not require expansion) from the innermost list and discard the rest.

```haskell
fibs = 0 : 1 : addLists (0 : 1 : addLists (0 : 1 : [1]) (1 : addLists (0 : 1 : [1]) (1 : addLists (0 : 1 : addLists (0 : 1 : [1]) (1 : addLists (0 : 1 : [1])

-- writing lists in common format (0 : 1 : [] == [0, 1])
fibs = 0 : 1 : addLists (0 : 1 : addLists [0, 1, 1] [1]) (1 : addLists [0, 1, 1] [1])

-- simplifying
fibs = 0 : 1 : addLists (0 : 1 : [1]) (1 : [1])

-- writing lists in common format
fibs = 0 : 1 : addLists ([0, 1, 1], [1, 1])

-- simplifying
fibs = 0 : 1 : [1, 2] --> [0, 1, 1, 2]
```

See? What we did was, we expanded `fibs` fully two times. And by discarding further expansions and simplifying, we added two new elements to our list. Note that we already began with 0 and 1.

We can easily write a small piece of code on top of this which returns the nth fibonacci number.

```haskell
fibs = 0 : 1 : addLists fibs (tail fibs)

fibonacci n = last $ take n fibs
```

Let's say `n = 30`. So it'll request 30 elements from `fibs`. Then, give us the last element of that 30 element list. i.e. the 30th element.

My biggest takeaway from this algorithm of fibonacci was that I need some time to get easy with infinite lists. I am used to approaching recursion from top-down. That means, start recursing and stop on some condition to yield result. That's how our naive approach works too.

This code does the opposite. It starts from 0 and never stops (theoretically). It will never reach a last element. The reason we're able to get away with writing this is that Haskell is lazy. It will only execute code if it really needs to. This is a huge departure from the strict evaluation that I'm used to.

So when we do a `take 30 fibs`, it'll start recursing. And since since we told it to actually give us 30 elements, it will start simplifying too. It allows us to extract elements from its front as it goes on building that list further and further. Once it has given us enough elements, it gives up calculating more. It simply isn't fussed about actually completing the list of all fibonacci numbers, in other words. Interesting, right?

While I know enough about recursion and Haskell library functions to try and explain how and why this code works, I imagine it'd take a bit of time for me to come up with such solutions myself. Hopefully sooner than later.

Let me know your thoughts over at [reddit thread](https://www.reddit.com/r/haskell/comments/43imla/haskell_diary_1_recursion/) for this post.

**Update 1**:
As */u/twistier* pointed out over at reddit, a better definition of recursion would be *a value, which may or may not be a function, being self referential*.
