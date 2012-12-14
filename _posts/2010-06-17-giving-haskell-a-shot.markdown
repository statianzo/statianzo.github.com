---
layout: post
title: Giving Haskell A Shot
---

[haskellhome]: http://haskell.org
As a typical C# developer, occasionally it's good to take a step out of the
Microsoft box, and see what other fascinating things exist. Let's take a look at
a programming language named [Haskell][haskellhome].


Hello, Haskell
---

Haskell is a [functional programming][functional] language. It's designed to be
use pure functions, meaning it's functions don't have side-effects. No objects
are modified, no global state changed. Whenever the same input goes into a
function, the same result will come out every time.

    add x y = x + y

Take the `add` function for example. Whenever you give it 2 and 3 as arguments,
it will always return 5.

I know, what you're thinking, "I can do that in language X already!" And you're
right, you can. However, this is just a simple example. Try this:

    prepend x ys = x : ys

The `prepend` method will place an item *x* on the front of a list *ys*. So for
example if you called `prepend 5 [3,2,1]` then `[5,3,2,1]` would be returned.
Different from many imperative languages, Haskell will not actually modify the
list *ys*, but rather create a new one with *x* at the head of it.

Lazy Evaluation
---

Haskell is lazy, in a good way. When performing list operations, it will wait
until the moment it needs to evaluate something before actually doing it. Say
for example you wanted to calculate the first 5 odd numbers:

    odds = [1,3..]
    take 5 odds

    --OUTPUT: [1,3,5,7,9]

The expression `[1,3..]` is a list starting at 1, incrementing by 2, that
continues on to infinity (assuming no memory/time/processor restrictions). `take
5` will return the first 5 values off of a list. If you wanted to get the first
10, 100, or 1000, just replace the *5* in `take 5`. 

For a slightly more complex example, consider getting the first 10  odd numbers
that are not a multiple of 5.

    oddsNotDivisibleBy5 = [x | x <- [1,3..], x `mod` 5 /= 0]
    take 10 oddsNotDivisibleBy5
    --OUTPUT: [1,3,7,9,11,13,17,19,21,23]

`oddsNotDivisibleBy5` is a *list comprehension*. It basically says, "give me *x*
where *x* is each odd number from 1 onward and meets the criteria of having a
non-zero modulus 5." `oddsNotDivisibleBy5` will continue indefinitely. However,
  the calculation of its values will be done when each is requested. 

Resources
---

Haskell can be quite a different world to plunge into. While there is no
Hitchhikers Guide, there are good resources available to get you on your way.

- [HaskellWiki][haskellhome]
- [Learn You A Haskell for Great Good][learnyou]
- [A Gentle Introduction to Haskell][gentle]
- [Real World Haskell][realworld]

[functional]: http://en.wikipedia.org/wiki/Functional_programming
[learnyou]: http://learnyouahaskell.com/
[gentle]: http://www.haskell.org/tutorial/ 
[realworld]: http://book.realworldhaskell.org/ 
