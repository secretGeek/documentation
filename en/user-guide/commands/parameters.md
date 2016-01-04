# Parameters (`TParam`)

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
