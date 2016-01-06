# Commands

Commands encapsulate logic to be executed in response to some user action. Examples of actions are clicking a **Save** menu item, tapping a phone icon, or stretching an image. Respectively, the associated logic might be to save outstanding changes, instigate a phone call, or zoom into the image.

Commands may or may not be executable in a given situation. For example, the command backing the **Save** menu item might be unavailable if there are no unsaved changes.

In MVVM, the view model does not concern itself with *how* a user executes a particular command - that decision is left to the view. But typically it's best if the execution logic resides in the view model so that changes to the view model's state can be encapsulated. Commands facilitate this separation between the *how* and the *what*.

If you've done any UI development in .NET, you're likely familiar with [the `ICommand` interface](https://msdn.microsoft.com/en-us/library/system.windows.input.icommand.aspx). It is an abstraction that can be used for exactly the purposes described. Its `CanExecute` method can be used to determine whether a command can be executed. If so, one can then invoke the `Execute` method. The `CanExecuteChanged` event can be used by interested parties to know when the command's executability has changed. For example, a `Button` bound to a particular `ICommand` would want to update its enabled state based on the executability of the command.

But the `ICommand` interface isn't an ideal abstraction. It fails to elegantly accommodate long-running commands, such as those that perform I/O. Moreover, its interface is imperative, not reactive. This makes it far less amenable to a reactive code base.

## Reactive Commands

Reactive commands also encapsulate logic to execute in response to user actions, but they do so via a reactive API. For example, instead of querying the command for its executability, you can use the `CanExecute` property, which is of type `IObservable<bool>`. Similarly, an `IsExecuting` property (also of type `IObservable<bool>`) tells you whether the command is currently executing, which could be a useful trigger to enable an activity animation, for example. And when you execute the command, you get back an `IObservable<TResult>`, where `TResult` is the "result" of executing your command (often just `Unit` if your command doesn't need to return anything of importance).

The fact that executing a command returns an observable of the result and not the result itself makes it clear that reactive commands are inherently asynchronous. You can kick off a CPU- or I/O-bound command without blocking your UI. Once completed, the result ticks through the observable and your UI can respond accordingly. Whilst the command is executing, it is unavailable (i.e. `CanExecute` will tick `false`). Thus, any UI elements bound to that command will automatically disable themselves whilst the command is executing.

Finally, reactive commands are themselves observable. Whenever any execution of the command completes, its value is observable by subscribing directly to the command.

> **Hint** All reactive commands also implement `ICommand`, but do so explicitly. This is to encourage the use of the reactive APIs rather than the imperative ones provided by `ICommand`. However, it also allows reactive commands to integrate seamlessly into frameworks that work against `ICommand`.

## An Example

Do we need a "compelling example"?

At what point do we drop directly into the meat/concepts instead of lots of text - maybe this is the place - we just call it something else and put in a diagram which shows the following high level:

IObservable canExecute = Observable.Return(true);
ReactiveCommand search = ReactiveCommand.Execute(canExecute, () => { return "Hello World"; });

search.Subscribe() -> results of the operation
search.ThrownExceptions() -> exceptions of the operation.

Then go onto async vs sync, canExecute etc.

It sets people up for what is about to come next.


## Asynchronous versus Synchronous Commands

Even though the API presented by `ReactiveCommand` is asynchronous, you are not required to perform your execution logic asynchronously. If your command is not CPU-intensive or I/O-bound then it probably makes sense to provide synchronous execution logic. You can do so by creating a command via `ReactiveCommand.Create`:

```cs
var command = ReactiveCommand.Create(() => Console.WriteLine("a synchronous reactive command));
```

There are several overloads of `Create` to facilitate commands that take parameters or return interesting values when they execute. These will be discussed in more detail below.

If, on the other hand, your command's logic *is* CPU- or I/O-bound, you'll want to use `CreateAsyncObservable` or `CreateAsyncTask`:

```cs
// here we're using observables to model asynchrony
var command1 = ReactiveCommand.CreateAsyncObservable(() => Observable.Return(Unit.Default).Delay(TimeSpan.FromSeconds(3)));

// here we're using the TPL to model asynchrony
var command2 = ReactiveCommand.CreateAsyncTask(async () =>
    {
        await Task.Delay(TimeSpan.FromSeconds(3)); 
    });
```

Again, several overloads exist for commands taking parameters and returning values.

Regardless of whether your command is synchronous or asynchronous in nature, you execute it via the `ExecuteAsync` method. You get back an observable that will tick the command's result value (which will just be `Unit.Default` if your command doesn't return anything of interest) when execution completes. Synchronous commands will execute _immediately_, so the observable you get back will already have completed. It is behavioral though, so subscribing after the fact will still tick through the result value. Moreover, this implies that there is no need to subscribe to the returned observable if you don't want/need the result. Often you will instead subscribe to the command itself and ignore the observable for individual executions of the command. In fact, it's normally the binding infrastructure that executes the command (discussed below).

## Command Parameters

Optionally, your command's execution logic can take a parameter. To do this, you need only use an appropriate overload of `Create*` when creating your `ReactiveCommand`:

```cs
// synchronous command taking a parameter
var command1 = ReactiveCommand.Create<int>(param => Console.WriteLine("Received parameter with type {0}: {1}.", param.GetType().Name, param);
// this outputs "Received parameter with type Int32: 42"
command1.ExecuteAsync(42);

// asynchronous command taking a parameter
var command2 = ReactiveCommand.CreateAsyncObservable<int>(param => Observable.Return(param).Do(p => Console.WriteLine("Received parameter with type {0}: {1}.", p.GetType().Name, p)));
// this outputs "Received parameter with type Int32: 42"
command2.ExecuteAsync(42);
```

The parameter's type is captured as `TParam` in `ReactiveCommand<TParam, TResult>`. The type of both `command1` and `command2` above is `ReactiveCommand<int, Unit>`.

Generally, you should avoid using command parameters. It is usually more appropriate for your view model to define properties for any state that your command's execution logic relies on.

## Command Values

The execution of a command produces a value, which is captured as `TResult` in `ReactiveCommand<TParam, TResult>`. If you don't need to return anything special, you can just use `Unit` for `TResult`. Indeed, this is what will happen automatically if you use `Create*` overloads that don't return values:

```cs
// a synchronous command that does not return an interesting result
var command1 = ReactiveCommand.Create(() => { });

// an observable-based asynchronous command that does not return an interesting result
var command2 = ReactiveCommand.CreateAsyncObservable(() => Observable.Return(Unit.Default));

// a Task-based asynchronous command that does not return an interesting result
var command3 = ReactiveCommand.CreateAsyncTask(async () => await Task.Delay(TimeSpan.FromSeconds(2)));
```

All the above commands are of type `ReactiveCommand<Unit, Unit>`. Note that `CreateAsyncObservable` is required to eventually return an `IObservable<T>`, so `T` must be known. Therefore, to achieve the same behavior we have to ensure our observable is of type `IObservable<Unit>`.

If we _do_ want to return something interesting each time our command executes, we need only use the appropriate `Create*` method:

```cs
// a synchronous command that always returns 42 upon execution
var command1 = ReactiveCommand.Create(() => 42);

// an observable-based asynchronous command that always returns 42 upon execution
var command2 = ReactiveCommand.CreateAsyncObservable(() => Observable.Return(42));

// a Task-based asynchronous command that always returns 42 upon execution
var command3 = ReactiveCommand.CreateAsyncTask(() => Task.FromResult(42));
```

Here, all commands are of type `ReactiveCommand<Unit, int>`. Subscribing to the observable return by `ExecuteAsync` (or subscribing to the command itself) will tick through the value `42`.

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

Here, `command` has additional constraints on its executability. Namely, both `UserName` and `Password` must be provided. Of course, `command` is still unavailable during execution of `LogOnAsync`, even if `UserName` and `Password` are both currently valid. In other words, `canExecute` *supplements* the default executability behavior; it doesn't replace it.

> **Warning** For performance reasons, `ReactiveCommand` does not marshal your `canExecute` observable to the main scheduler. You almost certainly want your `canExecute` observable to be ticking on the main thread, so be sure to add a call to `ObserveOn` if necessary.

## Controlling Scheduling

By default, `ReactiveCommand` uses `RxApp.MainThreadScheduler` to surface events. That is, values from `CanExecute`, `IsExecuting`, and result values from the command itself. Typically UI components are subscribed to these observables, so it's a sensible default. However, when writing unit tests for your view models, you may want more control over scheduling.

> **Warning** It's important to understand that the execution logic for a reactive command is *not* scheduled to execute on the provided scheduler (just as was the case for any `canExecute` observable you provide). Instead, it is left to the caller to implement any required scheduling inside their execution pipeline. This means it is entirely possible for your execution logic to execute on a thread other than that owned by the provided scheduler:
>
> ```cs
> var command = ReactiveCommand.Create(() => Console.WriteLine(Environment.CurrentManagedThreadId), scheduler: RxApp.MainThreadScheduler);
>
> // this will output the ID of the thread from which you make this call, not necessarily the ID of the main thread!
> command.ExecuteAsync();
> ```

All `Create*` methods take an optional `scheduler` parameter, so you can pass in a custom scheduler if you need to:

```cs
var command = ReactiveCommand.Create(() => {}, scheduler: someScheduler);
```

> **Note** If you're using ReactiveUI's `With` extension method in your tests, you can create commands using the default scheduling behavior. That's because the `With` extension method will switch out `RxApp.MainThreadScheduler` with the scheduler you provide it.

## Command Errors

If the logic you provide to a `ReactiveCommand` can fail in expected ways, you need a means of dealing with those failures.

For command execution, the pipeline you get back from `ExecuteAsync` will tick any errors that occur in your execution logic. However, the caller is under no obligation to subscribe to this pipeline and they typically will not choose to do so. Moreover, the caller is often a view and not well-suited to understanding and handling such errors.

To address this dilemma, `ReactiveCommand` includes a `ThrownExceptions` observable (of type `IObservable<Exception>`). Any errors that occur in your execution logic will *also* tick through this observable. If you haven't subscribed to it, ReactiveUI will bring down your application. This forces you towards a pit of error-handling success.

It can be tempting to *always* add a subscription to `ThrownExceptions`, even if the only recourse is to just log the problem. However, it is advisable to treat this like any other exception handling and only handle problems you can redress. If, for example, your command merely updates a property in your view model and it should never fail, any subscription to `ThrownExceptions` will serve only to obscure implementation problems. That said, be aware of the potential for intermittent problems, such as network and I/O errors. As always, a strong suite of tests will help you identify where a subscription to `ThrownExceptions` makes sense.

> **Note** Your `canExecute` pipeline also has the potential to produce an error. Such cases are almost certainly a programmer error because you never want your `canExecute` pipeline to end in error. Even so, such errors will also tick through `ThrownExceptions`.

## Binding


## Invoking Commands

At times it can be convenient to execute a command in response to some observable that isn't perhaps tied to a user interaction. For example, a feature that automatically saves the current document by executing a `SaveCommand` every 5 minutes. The `InvokeCommand` extension makes it easy to achieve this:

```cs
var interval = TimeSpan.FromMinutes(5);
Observable
.Timer(interval, interval)
.InvokeCommand(this.ViewModel, x => x.SaveCommand);
```

> **Hint** `InvokeCommand` respects the command's executability. That is, if the command's `CanExecute` method returns `false`, `InvokeCommand` will not execute the command when the source observable ticks.

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
