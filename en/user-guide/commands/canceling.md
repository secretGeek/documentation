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

