# Commands

Commands encapsulate logic to be executed in response to some user action. Examples of actions are clicking a **Save** menu item, tapping a phone icon, or stretching an image. Respectively, the associated logic might be to save outstanding changes, instigate a phone call, or zoom into the image.

Commands may or may not be executable in a given situation. For example, the command backing the **Save** menu item might be unavailable if there are no unsaved changes.

In MVVM, the view model does not concern itself with *how* a user executes a particular command - that decision is left to the view. But typically it's best if the execution logic resides in the view model so that changes to the view model's state can be encapsulated. Commands facilitate this separation between the *how* and the *what*.

If you've done any UI development in .NET, you're likely familiar with [the `ICommand` interface](https://msdn.microsoft.com/en-us/library/system.windows.input.icommand.aspx). It is an abstraction that can be used for exactly the purposes described. It's `CanExecute` method can be used to determine whether a command can be executed. If so, one can then invoke the `Execute` method. The `CanExecuteChanged` event can be used by interested parties to know when the command's executability has changed. For example, a `Button` bound to a particular `ICommand` would want to update its enabled state based on the executability of the command.

But the `ICommand` interface isn't an ideal abstraction. It fails to elegantly accommodate long-running commands, such as those that perform I/O. Moreover, its interface is imperative, not reactive. This makes it far less amenable to a reactive code base.

## An Example

Do we need a "compelling example"?

At what point do we drop directly into the meat/concepts instead of lots of text - maybe this is the place - we just call it something else and put in a diagram which shows the following high level:

IObservable canExecute = Observable.Return(true);
ReactiveCommand search = ReactiveCommand.Execute(canExecute, () => { return "Hello World"; });

search.Subscribe() -> results of the operation
search.ThrownExceptions() -> exceptions of the operation.

Then go onto async vs sync, canExecute etc.

It sets people up for what is about to come next.




## Controlling Executability

Reactive commands automatically become unavailable whilst they're executing. That is, their `CanExecute` observable will tick `false` when execution commences, and then `true` once execution completes. However, there are times you will want to further control a command's executability. For example, you might want a login command to be disabled until the user has entered both a user name and a password.

To do this, you can pass in an `IObservable<bool>` for the `canExecute` parameter when you create the command:

```cs
var canExecute = this
    .WhenAnyValue(
        x => x.UserName,
        x => x.Password,
        (u, p) => !string.IsNullOrEmpty(u) && !string.IsNullOrEmpty(p));
var command = ReactiveCommand.CreateAsyncObservable(this.LogOnAsync, canExecute);
```

Here, `command` has additional constraints on its executability. Namely, both `UserName` and `Password` must be provided. Of course, `command` is still unavailable during execution of `LogOnAsync`, even if `UserName` and `Password` are both currently valid. In other words, `canExecute` supplements the default executability behavior; it doesn't replace it.

> **Warning** `ReactiveCommand` does not marshal your `canExecute` observable to the main scheduler. You almost certainly want your `canExecute` observable to be ticking on the main thread, so be sure to add a call to `ObserveOn` if necessary.

## Controlling Scheduling

By default, `ReactiveCommand` uses `RxApp.MainThreadScheduler` to surface events. That is, values from `CanExecute`, `IsExecuting`, and result values from the command itself. Typically UI components are subscribed to these observables, so it's a sensible default. However, when writing unit tests for your view models, you may want more control over scheduling.

All `Create*` methods take an optional `scheduler` parameter, so you can pass in a custom scheduler if you need to:

```cs
var command = ReactiveCommand.Create(() => {}, scheduler: someScheduler);
```

> **Warning** it's important to understand that the execution logic for a reactive command is *not* scheduled to execute on the provided scheduler. Instead, it is left to the caller to implement any required scheduling inside their execution pipeline. This means it is entirely possible for your execution logic to execute on a thread other than that owned by the provided scheduler:
>
> ```cs
> var command = ReactiveCommand.Create(() => Console.WriteLine(Environment.CurrentManagedThreadId), scheduler: RxApp.MainThreadScheduler);
>
> // this will output the ID of the thread from which you make this call, not necessarily the ID of the main thread!
> command.ExecuteAsync();
> ```

> **Note** if you're using ReactiveUI's `With` extension method in your tests, you can create commands using the default scheduling behavior. That's because the `With` extension method will switch out `RxApp.MainThreadScheduler` with the scheduler you provide it.

## Command Errors

If the logic you provide to a `ReactiveCommand` can fail in expected ways, you need a means of dealing with those failures.

For command execution, the pipeline you get back from `ExecuteAsync` will tick any errors that occur in your execution logic. However, the caller is under no onus to subscribe to this pipeline and they typically will not choose to do so. Moreover, the caller is often a view and not well-suited to understanding and handling such errors.

To address this dilemma, `ReactiveCommand` includes a `ThrownExceptions` observable (of type `IObservable<Exception>`). Any errors that occur in your execution logic will *also* tick through this observable. If you haven't subscribed to it, ReactiveUI will bring down your application. This forces you towards a pit of error-handling success.

It can be tempting to *always* add a subscription to `ThrownExceptions`, even if the only recourse is to just log the problem. However, it is advisable to treat this like any other exception handling and only handle problems you expect. If, for example, your command merely updates a property in your view model and it should never fail, any subscription to `ThrownExceptions` will serve only to obscure implementation problems. That said, be aware of the potential for intermittent problems, such as network and I/O errors. As always, a strong suite of unit tests will help you identify where a subscription to `ThrownExceptions` makes sense.

> **Note** your `canExecute` pipeline also has the potential to produce an error. Such cases are almost certainly a programmer error because you never want your `canExecute` pipeline to end in error. Even so, such errors will also tick through `ThrownExceptions`.

## Binding

## Invoking Commands

## Combined Commands



# Misc


    paulcbetts [10:41 AM]
    It's not obvious that you should use Commands for things that aren't buttons in ReactiveUI, it's kind of a unique thing

    paulcbetts [10:41 AM]
    But any time you want to run something in the background, then bring its result back, ReactiveCommand is pretty cool

    michaelteper [11:20 AM] 
    I didnt know about `InvokeCommand`. Is that equivalent to `.Subscribe(_ => cmd.Execute(null))`?

    rdavisau [11:21 AM] 
    ^ it passes the value of the observable as the parameter, I believe

    michaelteper [11:22 AM] 
    that would make sense, sure

    michaelteper [11:22 AM]
    so `.Subscribe(x => cmd.Execute(x))`

    rdavisau [11:22 AM] 
    yep

    michaelteper [11:29 AM] 
    that would be good to know indeed

    ghuntley [11:30 AM] 
    would be kinda cool to see a fully fleshed out sample of your navigation pattern if you can afford the time :simple_smile:
    NEW MESSAGES

    paulcbetts [11:43 AM] 
    InvokeCommand respects CanExecute


One of the core goals of every MVVM library is to provide an implementation of
`ICommand`. This interface represents one of the two major parts of what
constitutes a ViewModel - Properties and Commands. In keeping with the goal of
MVVM + Rx, ReactiveUI provides its own implementation of `ICommand` called
`ReactiveCommand`, that works a bit differently than most other
implementations.

Commands represent discrete actions that are taken in the UI - "Copy", "Open",
and "Ok" are good examples of Commands. Usually these Commands are bound to a
control that is built to handle Commands, like a Button. Cocoa represents this
concept via the [Target Action
Framework](https://developer.apple.com/library/ios/documentation/general/conceptual/CocoaEncyclopedia/Target-Action/Target-Action.html).

Many Commands are invoked directly by the user, but some operations are also
useful to model via Commands despite being primarily invoked programatically.
For example, many code paths involving periodically loading or refreshing
resources (i.e. "LoadTweets") can be modeled well using Commands.
