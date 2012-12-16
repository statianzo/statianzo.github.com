---
layout: post
title: "ActionContainer: a Dynamic C# Service Agent"
---

ActionContainer is an implementation of the Service Agent pattern in
written in C#. It takes advantage of the C# 4.0 *dynamic* feature to eliminate
cluttering your code with empty message classes and also manipulate calls at
runtime.

Usage
---

Usage of ActionContainer is straight forward. Start by creating a class that
implements the empty `IActionProvider` interface with public methods.

{% highlight csharp %}
public class SimpleProvider : IActionProvider
{
  public void SayHello(string name)
  {
    Console.WriteLine("Hello, {0}", name);
  }

  public string GeneratePassword()
  {
    return "RaNdOmPaSsWoRd";
  }
}
{% endhighlight %}

Inject an instance of the ServiceAgent class, and make calls to the registered methods
through the `ServiceAgent` object.

{% highlight csharp %}
public class Needy : IDependOnSomething
{
    public dynamic ServiceAgent { get; set; }

    public void DoSomething()
    {
        ServiceAgent.SayHello("Buddy");
        string password = ServiceAgent.GeneratePassword();
    }
}
{% endhighlight %}

Calling `SayHello` with a string, will look up a registered method with the name
*SayHello*, takes a string argument, and has a void return type. Same behavior
goes for `GeneratePassword`, except it returns a string.

Looking up methods by name and parameters doesn't require creating a
class (think *SayHelloRequest*) for each registered action.

The ServiceAgent is also return type aware. In the call `string password =
ServiceAgent.GeneratePassword();`, a search is done for the GeneratePassword
that returns string. In this case, the `SimpleProvider` has the only registered
GeneratePassword. Lets say we added another provider, the `NumericProvider` as
follows:

{% highlight csharp %}
public class NumericProvider: IActionProvider
{
  public int GeneratePassword()
  {
    return 42;
  }
}
{% endhighlight %}

Now calls to `ServiceAgent.GeneratePassword();` can handle both int and string
return types with no special *PasswordResponse* or *NumericPasswordResponse*
wrapper classes.

{% highlight csharp %}
string strpwd = ServiceAgent.GeneratePassword(); //returns "RaNdOmPaSsWoRd"
int numpwd = ServiceAgent.GeneratePassword(); //returns 42
{% endhighlight %}

Additional Behavior
---

The default method lookup ability of ActionContainer is just that: the default.
ActionContainer provides a hook into its resolving process simply by creating
implementations of the `IActionListener` interface.

{% highlight csharp %}
public interface IActionListener
{
  bool CanHandle(ActionCallInfo callInfo);
  bool Handle(ActionListenerContext context);
}
{% endhighlight %}

The `CanHandle` method returns *true* if it has an opinion on the return value
of a call. If `CanHandle` is true, `Handle` allows the object to manipulate the
return value a value.

`ActionCallInfo` includes the method name, named parameters, unnamed
parameters, and expected return type. 

Using this information you could add helpful hooks such as a `LogListener`.


{% highlight csharp %}
public class LogListener : IActionListener
{
  public bool CanHandle(ActionCallInfo callInfo)
  {
    var arg = callInfo
      .NamedArguments
      .FirstOrDefault(
          x => x.Item1.Equals("log", StringComparison.OrdinalIgnoreCase)
      );
    if (arg == null || !(arg.Item2 is bool) || !(bool)arg.Item2)
      return false;
    var unnamed = callInfo.UnnamedArguments;
    Console.Write("Calling {0}({1}) - Args: ", callInfo.MethodName, callInfo.ReturnType);
    for (int i = 0; i < unnamed.Count; i++)
    {
      var val = unnamed[i];
      Console.Write("input{0}= {1} ", i, val);
    }
    Console.WriteLine();
    return false;

  }

  public bool Handle(ActionListenerContext context)
  {
    return false;
  }
}
{% endhighlight %}

The `LogListener` will watch for an argument named *log* with the value of
true. It will then write to the console the name of the call, return type, and
arguments.  To put it to use, just use the ServiceAgent as follows:

{% highlight csharp %}
ServiceAgent.SayHello("Buddy", log: true);
//Output:
//Calling SayHello() - Args: input0= Buddy
//Hello, Buddy
{% endhighlight %}

Because the ServiceAgent is a dynamic object, any number of named arguments can
be passed to its method calls.

Another possible hook would be a `StringTrimListener`. Any arguments that are strings
can be trimmed before getting passed to the handling method.

{% highlight csharp %}
public class StringTrimListener : IActionListener
{
  public bool CanHandle(ActionCallInfo callInfo)
  {
    var unnamed = callInfo.UnnamedArguments;
    for (int i =0; i < unnamed.Count; i++)
    {
      var val = unnamed[i] as string;
      if (val != null)
        unnamed[i] = val.Trim();
    }
    return false;
  }

  public bool Handle(ActionListenerContext context)
  {
    return false;
  }
}
{% endhighlight %}

No change in code is needed to have calls to the ServiceAgent start using the
`StringTrimListener`. Just make a call on something with a string with hanging
spaces.

{% highlight csharp %}
ServiceAgent.SayHello("       Friend     ", log: true);
//Output:
//Hello, Friend
{% endhighlight %}

Things To Keep In Mind
---

ActionContainer has some requirements that you must keep in mind while using it.
Most deal with determining types.

First, when doing variable assignment you must explicitly provide the type when
returning values. Using `var` will result in unfriendly behavior.

{% highlight csharp %}
var result = ServiceAgent.GeneratePassword(); //BAD!
{% endhighlight %}

Second, passing null values as parameters limits lookup. This has to do with
null not carrying any type information other than *object*.

{% highlight csharp %}
string name = null;
ServiceAgent.SayHello(name); //Less than favorable
{% endhighlight %}

Finally, ActionContainer is slow (~10x) compared to direct method calls. Using
it in a long running loop will have noticeable performance hits. However, in
response to user actions, incoming web request, etc, the difference is negligible. 

Getting ActionContainer
---

It's on [Github](http://github.com/statianzo/ActionContainer). Go and grab it with Git.

    $ git clone git://github.com/statianzo/ActionContainer.git


