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
