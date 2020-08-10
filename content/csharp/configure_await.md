---
title: "Configure Await"
date: 2020-08-09T18:09:50-04:00
---

Reading through a [asp.net core project](https://github.com/ivanpaulovich/clean-architecture-manga/tree/dotnet5), 
I came across number of calls that looked like this:

```cs
int affectedRows = await this._context
    .SaveChangesAsync()
    .ConfigureAwait(false); // ME: what does this do?
return affectedRows;
```

I dug into what this was actually doing by reading [this blog](https://devblogs.microsoft.com/dotnet/configureawait-faq/):

- `await` uses a custom callback strategy based on the calling context
- `ConfigureAwait(false)` ignores the custom strategy.

<!--more-->

First off, I just want to say that I *don't like this code*.
Code like `ConfigureAwait(false)` strewn around a codebase are not very expressive snippets - they don't convery much *developer intent* for what that method should actually be doing.
And, looking at the example codebase, the developers used this snippet *everywhere*, so I became curious as to why it wasn't the default behavior (and of course how it was actually different than the default behavior).

> Everything written beyond this point is my summary of [this blog](https://devblogs.microsoft.com/dotnet/configureawait-faq/) for the purposes of my own understanding.

## What does await do?

`Await` works by telling the compiler to convert code to [continuation-passing-style](https://en.wikipedia.org/wiki/Continuation-passing_style) - a fancy way of saying that the next lines of imperitive code become a *callback* or function to be executed once the current 'awaited' line finishes.
The general algorithm is to split the code at each `await` into a new continuation that gets execeuted when the async code completes.

Thus, code like this: 

```cs
var x = await ComputeValueAsync();
Console.WriteLine($"x is now {x}")
```

*roughly* becomes this (see [this blog](https://devblogs.microsoft.com/premier-developer/dissecting-the-async-methods-in-c/) for more accurate rendition):

```cs
ComputeValueAsync().ContinueWith(result => Console.WriteLine($"x is {result}"));
```


The only other component is *threading*.
Generally, async code is used to offload work from the current thread.
Imagine you (the current thread) are a factory worker with a bunch of minions (your thread pool).
When you need to do something tedious, say finding the name of your mother in law to send her a letter, you can ask your minions to do it for you (schedule a task in a thread pool).
This minion has two different response options:

- A) Once they have found your mother in law's name, they can tell you what it is so *you* can send the letter

    This would mean *returning the value to the original calling thread / context*.

- B) Once they have found your mother in law's name, they can send the latter (or tell someone else to send it)

    This would mean that execution could continue on *any* of the available calling contexts.

This is where configure await comes in.

## ConfigureAwait(false)

The decision made by the .NET core compiler between choice `A` and choice `B` is *generally* choice `B`: the compiler will continue executing the async code *without* forcing it back into the calling context.
But, there are certain times where it matters which context code is executed in, such as in a GUI.

In GUI programming, properties can only  be accessed safely in the UI thread.
So, simple code like `button.text = await http.GetStringAsync("http://example.com/")` cannot simply use strategy `A` shown above -
If the continuation, setting the button text, is run in a *different* context than the original UI context, the code will produce an error.

Luckily for application developers, the .NET application frameworks generally set the default behavior by setting the `SynchronizationContext` for you.
This can be done via the `SynchronizationContext.SetSynchronizationContext(myContext)` method.
The custom `myContext` object will be used to configure the strategy used to determine which execution context completes the continuation.
Ergo, WPF will *force* await calls to return their continuations to the calling context by `enqueueing` the continuation to the calling context.
Once the UI thread is able to pull the continuation off the queue, it will execute the code to set the button text and no exceptions will be thrown.


## Why not always use this custom SynchronizationContext?

Overhead and Deadlocks.

Forcing the continuation to be handled by the calling context *might* be needless, and it takes a bit of time to do so.
So, it adds possibly uneeded overhead.

More importantly, queueing the continuation into the calling context can create deadlocks.
Imagine a webserver makes a *synchronous* call to some webserver by calling an Async function and calling `.Wait()`,blocking the current thread until the Task completes.
Upon finishing the download, it enqueues the continuation to original context, but that thread cannot handle callback as it is blocked.
The thread will remain blocked until the callback is executed, so the system is in deadlock.

By adding `ConfigureAwait(false)`, which tells the compiler explicitly to NOT use the calling context to schedule callbacks, the continuation will not be entered onto the original queue, but rather some other thread in the thread pool
This will avoid the deadlock.

> Note: the original caller also should not have bocked on the async code, as that is the other requirement for this deadlock.
> See [Don't Block on Async Code](https://blog.stephencleary.com/2012/07/dont-block-on-async-code)
> But, doing both makes doubly sure you don't deadlock your application.


## Read More

1. [Clean Architecture Manga](https://github.com/ivanpaulovich/clean-architecture-manga/tree/dotnet5). 
1. [Configure Await FAQ](https://devblogs.microsoft.com/dotnet/configureawait-faq/)
1. [Continuation Passing Style](https://en.wikipedia.org/wiki/Continuation-passing_style) 
1. [Dissecting the async methods in C#](https://devblogs.microsoft.com/premier-developer/dissecting-the-async-methods-in-c/) 
1. [Don't Block on Async Code](https://blog.stephencleary.com/2012/07/dont-block-on-async-code)