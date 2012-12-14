---
layout: post
title: Using Yield in C#
---

yield basics
---

yield places values into an IEnumerable
object. To further understand what yield does, here's an example with trace
statements between each yield statement:

    //:SimpleYield.cs
    using System;
    using System.Collections;

    internal class SimpleYield
    {
      private static IEnumerable GetValues()
      {
        Console.WriteLine("Returning 1");
        yield return 1;
        Console.WriteLine("Returning 2");
        yield return 2;
        Console.WriteLine("Returning 3");
        yield return 3;
      }

      private static void Main()
      {
        //Iterate over YieldMethod's IEnumerable
        foreach (int i in GetValues())
          Console.WriteLine(i);
      }
    }

    /*Output:
    Returning 1
    1
    Returning 2
    2
    Returning 3
    3 */ //: 

GetValues( )'s yield
statements return hard-coded integer values. Main( ) iterates these
values, printing each. Notice how GetValue( )'s trace statements
interlace with Main( )'s. A yield statement comes in two different
forms, yield return and yield break. yield return places the evaluated
expression as the current value of the IEnumerable. yield break marks the
end of an iterator:

    //:YieldBreak.cs
    using System;
    using System.Collections;

    internal class YieldBreak
    {
      private static IEnumerable GetValues()
      {
        Console.WriteLine("Returning 1");
        yield return 1;
        Console.WriteLine("Returning 2");
        yield return 2;
        Console.WriteLine("Breaking");
        yield break; //Break Here
        Console.WriteLine("Returning 3");
        yield return 3;
      }

      private static void Main()
      {
        foreach (int i in GetValues())
          Console.WriteLine(i);
      }
    }

    /*Output:
    Returning 1
    1
    Returning 2
    2
    Breaking */ //:

yield break prevents remaining code from executing.
It reports back to the foreach that
there are no values remaining in the IEnumerable. C#'s compiler reports a
warning about any unreachable code following yield break.

**Exercise 1:**
Use Visual Studio's debugger to step through (F11) SimpleYield.cs.
Follow the code path as the foreach executes.

**Exercise 2:**
Create a GetValue( ) method that returns a random number of integer
values. Use yield return and yield break.

    //: Exercise2.cs
    using System;
    using System.Collections;

    internal class Exercise2
    {
      public static IEnumerable GetValues()
      {
        var rand = new Random();
        int current = 0;
        while (true)
        {
          current = rand.Next(20);
          if (current == 17)
            yield break;
          yield return current;
        }
      }

      public static void Main()
      {
        foreach (int i in GetValues())
          Console.WriteLine(i);
      }
    }

    //: 

yield Benefits
---

 One strong feature of yield is its ability to defer processing. It delays
any calculation until absolutely necessary. Deferred processing makes
more responsive programs. yield spreads wait time amongst all iterations
            by creating values only when necessary:

    ///: DeferredProcessingTime.cs
    using System;
    using System.Collections;
    using System.Threading;

    internal class
      DeferredProcessingTime
    {
      public static IEnumerable CalculateAtOnce()
      {
        var intArray = new int[10];
        for (int i = 0; i < 10; i++)
        {
          Thread.Sleep(1000);
          intArray[i] = i;
        }
        return intArray;
      }

      public
        static IEnumerable DeferredCalculate()
      {
        for (int i = 0; i < 10; i++)
        {
          Thread.Sleep(1000);
          yield return i;
        }
      }

      public static void Main()
      {
        Console.WriteLine("Calculate at once");
        //Ten seconds before printing
        foreach (int i in CalculateAtOnce())
          Console.Write(i);
        Console.WriteLine("\nUsing Yield");
        //One second between each write
        foreach (int z in DeferredCalculate())
          Console.Write(z);
      }
    }

    /*Output:
    Calculate at once
    0123456789
    Using Yield
    0123456789 */
    //:

The above example emulates an algorithm that takes a long time to process by
using Thread.Sleep( ) before adding each value. Both foreachs over
CalculateAtOnce( ) and DefferedCalculate( ) print the same results
to the console. However the execution behavior is noticeably
different. CalculateAtOnce( )'s foreach waits for 10 seconds and
prints 0 through 9. DeferredCalculate( )'s foreach prints each
value per second. Another benefit of yieldâ€™s deferred processing
is that it does not require a collection stored in memory. yield can
iterate over a large set of data without consuming large amounts of
memory. Take the following example. You iterate over a set of data
with 10 million integer values. Without yield, the entire collection
of values must be returned. With yield, each integer value is
returned when requested.

    ///:DeferredProcessingMemory.cs
    using System;
    using System.Collections;

    internal class DeferredProcessingMemory
    {
      public static IEnumerable
        CalculateAtOnce()
      {
        var intArray = new int[10000000];
        for (int i = 0; i < 10000000; i++)
          intArray[i] = i;
        return
          intArray;
      }

      public static IEnumerable DeferredCalculate()
      {
        for (int i = 0; i < 10000000; i++)
          yield return i;
      }

      public static void Main()
      {
        Console.WriteLine("Using Yield");
        IEnumerable deferred = DeferredCalculate();
        Console.ReadLine();
        //Memory usage: ~2MB Console.WriteLine("Calculating at once");
        IEnumerable atOnce = CalculateAtOnce();
        Console.ReadLine();
        //Memory usage: ~40MB
      }
    }

    /*Output:
    Using Yield
    Calculating at once 
    */ //: 

Start the Windows Task Manager
(Ctrl+Shift+Esc) before running this example. Find your process,
and notice the amount of memory usage from using
DeferredCalculate( ). Press enter and see the drastic increase
that CalculateAtOnce( ) makes; intArray's 10 million in-memory
values are the cause. yield gives potential to handle data sets
of any size without concern for memory limitations.

**Exercise 3:**
Use yield to iterate over an infinite data set of random integers.

    ///:Exercise3.cs
    using System;
    using System.Collections;

    internal class Exercise3
    {
      public static Random rand = new Random();

      public static IEnumerable GetValues()
      {
        while (true)
          yield return rand.Next();
      }

      public static void Main()
      {
        foreach (int i in
          GetValues()) Console.WriteLine(i);
      }
    }
    ///:

Under the Hood
---
Many C# keywords involve the
generation of MSIL code, methods, or even classes. yield
is no exception.
[Red Gate's .NET Reflector][reflector]
shows the generated C# from using the yield statement.
The disassembled assembly of SimpleYield.cs contains an extra class implementing
*IEnumerable*.

    public class GeneratedClass : IEnumerable,
                                  IEnumerator,
                                  IDisposable
    {
      //...CUT...
      public bool MoveNext()
      {
        switch (this.state)
        {
          case 0:
            this.state = -1;
            Console.WriteLine("Returning 1");
            this.current = 1;
            this.state = 1;
            return true;
          case 1:
            this.state = -1;
            Console.WriteLine("Returning 2");
            this.current = 2;
            this.state = 2;
            return true;
          case 2:
            this.state = -1;
            Console.WriteLine("Returning 3");
            this.current = 3;
            this.state = 3;
            return true;
          case
            3:
            this.state = -1;
            break;
        }
        return false;
      }

    //...CUT...
    }

The compiler generates a switch
statement to handle executing the code written in
your original yielding method. Only one case
statement is used for each time MoveNext( ) is
called. This is how the deferred processing
actually occurs. GetValues( ) is also changed to
use the GeneratedClass( ) in place of the original code.

      private static IEnumerable GetValues()
      {
        return new GeneratedClass(-2);
      } 

The -2 in
the new GeneratedClass( ) call is used to set
the initial state. The state's value is
changed when MoveNext( ) is called. Beginning
each case, state is set to -1 to prevent further
execution if an exception occurs. Just before a
case returns, state is assigned the value of the
next case.

**Exercise 4:**
Write an implementation of yield's generated code and execute a foreach over it.

    //:Exercise4.cs
    using System;
    using System.Collections;

    internal class Exercise4 :
      IEnumerable, IEnumerable,
      IEnumerator, IEnumerator, IDisposable
    {
      private string current = "";
      private int state;

      public string Current
      {
        get { return current; }
      }

      public void Dispose()
      {
      }

      IEnumerator IEnumerable.GetEnumerator()
      {
        return GetEnumerator();
      }

      object IEnumerator.Current
      {
        get { return Current; }
      }

      public bool MoveNext()
      {
        switch (state)
        {
          case 0:
            state = -1;
            current = "One";
            state = 1;
            return
              true;
          case 1:
            state = -1;
            current
              = "Two";
            state = 2;
            return true;
          case 2:
            state = -3;
            current = "Three";
            state = 3;
            return true;
        }
        return false;
      }

      public void Reset()
      {
        throw new NotImplementedException();
      }

      public IEnumerator GetEnumerator()
      {
        return this;
      }

      public static void Main()
      {
        foreach (string s in new Exercise4())
          Console.WriteLine(s);
      }
    }

    /*Output:
    One
    Two
    Three */ //: 

[reflector]: http://www.red-gate.com/products/reflector/
