# Interactions

At times you may find yourself writing view model code that needs to confirm something with the user. For example, checking if it's OK to delete a file, or asking what to do about an error that has occurred.

It might be tempting to simply throw up a message box right from within the view model. But that would be a mistake. Not only does this tie your view model to a particular UI technology, it also makes testing difficult (or even impossible).

Instead what is needed is a means of suspending the view model's execution path until some data is provided by the user. ReactiveUI's interaction mechanism facilitates just this.

## API Overview

Underpinning the user interaction infrastructure is the abstract `Interaction` class. This class provides a means of referring to _any_ interaction, regardless of the type of the interaction's result.

The `Interaction<TResult>` class extends `Interaction` and strongly-types the result of the interaction. For example, an `Interaction<bool>` is capable of obtaining a yes/no answer.

For richer interactions that require more than a typed result, you can subclass `Interaction<TResult>`. In fact, ReactiveUI provides one such subclass called `ErrorInteraction<TResult>`. This class adds an `Error` property (of type `Exception`), which makes it a suitable option in error recovery scenarios.

The interaction encapsulates only the result of the interaction, as well as any data that supports it. It does not provide a distribution or handling mechanism for interactions. For that, ReactiveUI provides the `InteractionBroker<TInteraction>` class.

Interaction brokers are the means by which collaborating components communicate. Normally this implies that both the view model and view are hooked into the same broker. The view registers a handler against the broker, which can be asynchronous. Then, the view model "raises" an interaction against the broker.

## An Example

```cs
public class ViewModel : ReactiveObject
{
    private readonly InteractionBroker<Interaction<bool>> confirm;
    
    public ViewModel()
    {
        this.confirm = new InteractionBroker<Interaction<bool>>();
    }
    
    public InteractionBroker<Interaction<bool>> Confirm => this.confirm;
    
    public async Task DeleteFileAsync()
    {
        var confirmation = new Interaction<bool>();
        
        // this will throw an exception if nothing handles the interaction
        await this.confirm.Raise(confirmation);
        
        if (confirmation.GetResult())
        {
            // delete the file
        }
    }
}

public class View
{
    public View()
    {
        this.WhenActivated(
            d =>
            {
                d(this
                    .ViewModel
                    .Confirm
                    .RegisterHandler(
                        async interaction =>
                        {
                            var deleteIt = await this.DisplayAlert(
                                "Confirm Delete",
                                "Are you sure you want to delete this super important file?",
                                "YES",
                                "NO");

                            interaction.SetResult(deleteIt);
                        }));
            });
    }
}
```

You can also create a shared broker that is utilized by multiple components in your application. A common example of this is in error recovery. Many components may want to raise errors, but we may want only one common handler. Here's an example of how you can achieve this:

```cs
public enum ErrorRecoveryOption
{
    Retry,
    Abort
}

public class AppErrorInteraction : ErrorInteraction<ErrorRecoveryOption>
{
    public AppErrorInteraction(Exception error)
        : base(error)
    {
    }
}

public static class InteractionBrokers
{
    public static readonly InteractionBroker<AppErrorInteraction> Errors = new InteractionBroker<AppErrorInteraction>();
}

public class SomeViewModel : ReactiveObject
{
    public async Task SomeMethodAsync()
    {
        while (true)
        {
            Exception failure = null;
            
            try
            {
                DoSomethingThatMightFail();
            }
            catch (Exception ex)
            {
                failure = ex;
            }
            
            if (failure == null)
            {
                break;
            }
            
            var recovery = new AppErrorInteraction(failure);
            
            // this will throw if nothing handles the interaction
            await InteractionBrokers.Errors.Raise(recovery);
            
            if (interaction.GetResult() == ErrorRecoveryOption.Abort)
            {
                break;
            }
        }
    }
}

public class RootView
{
    public RootView()
    {
        InteractionBrokers.Errors.RegisterHandler(
            async interaction =>
            {
                var action = await this.DisplayAlert(
                    "Error",
                    "Something bad has happened. What do you want to do?",
                    "RETRY",
                    "ABORT");

                interaction.SetResult(action ? ErrorRecoveryOption.Retry : ErrorRecoveryOption.Abort);
                        });
    }
}
```

> **Note** For the sake of clarity, the example code here mixes TPL and Rx code. Production code would normally stick with one or the other.

## Handler Precedence

The implementation of `InteractionBroker<TInteraction>` facilitates a handler chain. Any number of handlers can be registered, and later registrations are deemed of higher priority than earlier registrations. When an interaction is raised, each handler is given the _opportunity_ to handle that interaction (i.e. set a result). The handler is under no obligation to actually handle the interaction. If a handler chooses _not_ to set a result, the next handler in the chain is invoked.

This chain of precedence makes it possible to define default handlers at a root level of your application, then override those handlers from higher level components. For example, a root level handler may provide default error recovery behavior. But a specific view in the application may know how to recover from a certain error without prompting the user. It could register a handler whilst it's activated, then dispose of that registration when it deactivates. Obviously such an approach requires a shared interaction broker instance.

> **Note** The `InteractionBroker<TInteraction>` class is designed to be extensible. Subclasses can change the behavior of `Raise` such that it does not exhibit the behavior described above. For example, you could write an implementation that tries only the first handler in the list.

## Unhandled Interactions

Any interaction can go unhandled. If there are no handlers, or none of the handlers set a result, the interaction is itself is considered unhandled. This is always a programming error.

In this circumstance, the invocation of `Raise` will result in an `UnhandledInteractionException` being thrown. The underlying interaction is exposed via the `Interaction` property of this exception.

## Testing

You can easily test interaction logic in view models by hooking into the relevant interaction broker:

```cs
[Fact]
public async Task interaction_test()
{
    var fixture = new ViewModel();
    fixture
        .Confirm
        .RegisterHandler(interaction => interaction.SetResult(true));
        
    await fixture.DeleteFileAsync();
    
    Assert.True(/* file was deleted */);
}
```

If your test is hooking into a shared broker, you probably want to dispose of the registration:

```cs
[Fact]
public async Task interaction_test()
{
    var fixture = new SomeViewModel();
    
    using (InteractionsBrokers.Error.RegisterHandler(interaction => interaction.SetResult(ErrorRecoveryOption.Abort)))
    {
        fixture.SomeMethodAsync();
        
        // assert abort here
    }
}
```