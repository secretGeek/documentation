## Canceling

If your command's execution logic can take a long time to complete, it can be useful to allow the execution to be canceled. This cancellation support can be used internally by your view models, or exposed so that users have a say in the matter.

### Basic Cancelation

At its most primitive form, canceling a command's execution involves disposing the execution subscription:

```cs
var subscription = someReactiveCommand
    .Execute()
    .Subscribe();

// this cancels the command's execution
subscription.Dispose();
```

However, this requires you to obtain, and keep a hold of the subscription. If you're using bindings to execute your commands you won't have access to the subscription.

### Canceling via Another Observable

Rx itself has intrinsic support for canceling one observable when another observable ticks. It provides this via the `TakeUntil` operator:

```cs
var cancel = new Subject<Unit>();
var command = ReactiveCommand
    .CreateFromObservable(
        () => Observable
            .Return(Unit.Default)
            .Delay(TimeSpan.FromSeconds(3))
            .TakeUntil(cancel));
            
// somewhere else
command.Execute().Subscribe();

// this cancels the above execution
cancel.OnNext(Unit.Default);
```

Of course, you wouldn't normally create a subject specifically for cancellation. Normally you already have some other observable that you want to use as a cancellation signal. An obvious example is having one command cancel another:

```cs
public class SomeViewModel : ReactiveObject
{
    public SomeViewModel()
    {
        this.CancelableCommand = ReactiveCommand
            .CreateFromObservable(
                () => Observable
                    .Return(Unit.Default)
                    .Delay(TimeSpan.FromSeconds(3))
                    .TakeUntil(this.CancelCommand));
        this.CancelCommand = ReactiveCommand.Create(
            () => { },
            this.CancellableCommand.IsExecuting);
    }
    
    public ReactiveCommand<Unit, Unit> CancelableCommand
    {
        get;
        private set;
    }
    
    public ReactiveCommand<Unit, Unit> CancelCommand
    {
        get;
        private set;
    }
}
```

Here we have a view model with a command, `CancelableCommand`, that can be canceled by executing another command, `CancelCommand`. Notice how `CancelCommand` can only be executed when `CancelableCommand` is executing.

> **Note** At first glance there may appear to be an irresolvable circular dependency between `CancelableCommand` and `CancelCommand`. However, note that `CancelableCommand` does not need to resolve its execution pipeline until it is executed. So as long as `CancelCommand` exists before `CancelableCommand` is executed, the circular dependency is resolved.

### Cancellation with the Task Parallel Library

Cancellation in the TPL is handled with `CancellationToken` and `CancellationTokenSource`. Rx operators that provide TPL integration will normally have overloads that will pass you a `CancellationToken` with which to create your `Task`. The idea of these overloads is that the `CancellationToken` you receive will be canceled if the subscription is disposed. So you should pass the token through to all relevant asynchronous operations.

