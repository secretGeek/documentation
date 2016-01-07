# Asynchronous versus Synchronous Commands

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
