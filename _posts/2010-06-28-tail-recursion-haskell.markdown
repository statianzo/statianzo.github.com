---
layout: post
title: Tail Recursion In Haskell
---

[recursion]: http://en.wikipedia.org/wiki/Recursion_(computer_science)
Recursion
---

[Recursion][recursion] in its simplest form can be understood as a function that calls
itself.

    fact x = x * fact (x - 1)


In the above function, `fact(x)` is equal to `x` times the value of `fact(x-1)`.
`fact` can be described as infinitely recursive; it will never complete because
it doesn't have a *base case*. A *base case* could be something like `fact 1 =
1`, where the function no longer recurses and will be capable of terminating.

    fact 0 = 1
    fact x = x * fact (x - 1)

If `x` is larger than 0, `fact` will eventually terminate, and the factorial of
that number will be returned.

Tail Recursion
---

The term *tail recursion* refers to a form of recursion in which the final
operation of a function is a call to the function itself.

    fact2 x =
      tailFact x 1
      where tailFact 0 a = a
            tailFact n a = tailFact (n - 1) (n * a)

The `fact2` function wraps a call to `tailFact` a function that's tail
recursive. The first argument `n` in `tailFact` tells the function we want the
nth factorial. Second, `a`, is an accumulator that maintains the values of the
previous multiplication operations.

Why use Tail Recursion?
---

So even the simple examples make it obvious, tail recursion can come with some
added complexity. What good is it other than to confuse other readers of your
code? The answer has to do with how most programming languages handle function
calls. Take this small example:

    foo x = 10 + x 
    bar = 3 * foo 4

Say your program is in function `bar` and it reaches the call to `foo`. It then
proceeds to execute the code at the memory address of the `foo` function. When
`foo` completes, how does your program know where to go back to? The *call
stack*, of course.

Generally, the *call stack* is a structure in memory that tracks the current
depth of a running program. As functions call other functions, the number of
locations on the stack grows. You can think of it as digital breadcrumbs.

As with any memory structure, there is a limit to how large the call stack can
grow. It's large enough to not worry about most of the time. However, when doing
recursion, the number of items on the stack can rise quickly. Running out of
room can result in a *stack overflow*, which will likely terminate your program
or at least not give you the result you expected.

Assuming a language's compiler can optimize for it, Tail recursion can help
overcome this issue. Because there are no hanging operations left in the
function, the same stack frame can be reused. That's why an accumulator was
needed in the `tailFact` function; it eliminates having to multiply after the
recursive call.

Another Example: Fibonacci Numbers
---

A popular place for using recursion is calculating [Fibonacci numbers][fib].
They are part of a sequence as follows: 1,2,3,5,8,13,21... Starting at 1, each
term of the Fibonacci sequence is the sum of the two numbers preceding it. A
simple recursive solution in Haskell is as follows:

    fibs 0 = 1
    fibs 1 = 1
    fibs n = fibs (n - 1) + fibs (n - 2)

Notice that the `fibs` function needs to call itself twice to calculate the nth
Fibonacci. The number of recursive calls grows exponentially where the first two
calls will each make two of their own, and so on.

Using tail recursion, while slightly more complex, will prevent the exponential
growth of function calls.

    fibs2 = tailFibs 0 1 0 

    tailFibs prev1 prev2 start end 
              | start == end = next
              | otherwise = tailFibs next prev1 (start + 1) end
              where next = prev1 + prev2

The workhorse in this solution is `tailFibs`, takes four arguments, three are
accumulators of some sort. `prev1` and `prev2` are the previous first and second
terms respectively. Start is the index of the currently calculated term, and end
is passed through on each call for the base condition.

In this instance, tail recursion turns out to be much more performant over the
simpler implementation. Attempting to get even the 100th term shows a
significant difference between them.

Use Appropriately
---

Being able to approach problems with the power of tail recursion can help
conquer in a way that feels natural, without mutable state, and without worry of
stack overflows. It's part of what makes functional languages like Haskell
shine.

Tail recursion, while useful, is best used for algorithms that are recursive in
nature and likely to wind up going very deep. Otherwise, you may wind up with
something far more complex and fragile than necessary.

[fib]: http://en.wikipedia.org/wiki/Fibonacci_number
