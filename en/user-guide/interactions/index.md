# Interactions

At times you may find yourself writing view model code that needs to confirm something with the user. For example, checking if it's OK to delete a file, or asking what to do about an error that has occurred.

It might be tempting to simply throw up a message box right from within the view model. But that would be a mistake. Not only does this tie your view model to a particular UI technology, it also makes testing difficult (or even impossible).

Instead what is needed is a means of suspending the view model's execution path until some data is provided by the user. ReactiveUI's interaction mechanism facilitates just this.

## API Overview

Underpinning the interaction infrastructure is the `Interaction<TInteractionData>` class. This class provides the glue between collaborating components of the interaction. It is responsible for distributing interactions to handlers.

The `TInteractionData` generic type argument must be a type that extends the `InteractionData` class. An instance of `InteractionData` encapsulates the data related to an interaction, as well as the interaction result.

`InteractionData` is abstract, and is not something you'd typically use directly. Instead, you can use the `InteractionData<TResult>` class. This class strongly-types the result of the interaction. For example, if your interaction requires a yes/no answer, you could use an `InteractionData<bool>` to obtain it.

If your interaction requires extra data specific to your use case, you can subclass `InteractionData<TResult>`. In fact, ReactiveUI includes one such subclass called `ErrorInteractionData<TResult>`. This class adds an `Error` property (of type `Exception`), which makes it a suitable option in error recovery scenarios.

A typical arrangement of interaction components and their roles is:

* **View Model**: wants to know the answer to a question, such as "is it OK to delete this file?"
* **View**: asks the user the question, and supplies the answer during the interaction

Whilst this configuration is the most common, it is by no means required. You could, for example, have the view answer the question itself without user intervention. Or perhaps both components are view models. The interactions infrastructure provided by ReactiveUI does not place any restrictions on collaborating components.

Assuming the common configuration, a view would register a handler by calling one of the `RegisterHandler` methods on `Interaction<TInteractionData>` instance. The view model would pass in an `InteractionData` instance to the `Handle` method.

## An Example

```cs
public class ViewModel : ReactiveObject
{
    private readonly Interaction<InteractionData<bool>> confirm;
    
    public ViewModel()
    {
        this.confirm = new Interaction<InteractionData<bool>>();
    }
    
    public Interaction<InteractionData<bool>> Confirm => this.confirm;
    
    public async Task DeleteFileAsync()
    {
        var confirmation = new InteractionData<bool>();
        
        // this will throw an exception if nothing handles the interaction
        await this.confirm.Handle(confirmation);
        
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

You can also create an `Interaction<TInteractionData>` that is shared across multiple components in your application. A common example of this is in error recovery. Many components may want to raise errors, but we may want only one common handler. Here's an example of how you can achieve this:

```cs
public enum ErrorRecoveryOption
{
    Retry,
    Abort
}

public class AppErrorInteractionData : ErrorInteractionData<ErrorRecoveryOption>
{
    public AppErrorInteractionData(Exception error)
        : base(error)
    {
    }
}

public static class Interactions
{
    public static readonly Interaction<AppErrorInteractionData> Errors = new Interaction<AppErrorInteractionData>();
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
            
            var recovery = new AppErrorInteractionData(failure);
            
            // this will throw if nothing handles the interaction
            await Interactions.Errors.Handle(recovery);
            
            if (recovery.GetResult() == ErrorRecoveryOption.Abort)
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
        Interactions.Errors.RegisterHandler(
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

`Interaction<TInteractionData>` implements a handler chain. Any number of handlers can be registered, and later registrations are deemed of higher priority than earlier registrations. When an interaction is passed into the `Handle` method, each handler is given the _opportunity_ to handle that interaction (i.e. set a result). The handler is under no obligation to actually handle the interaction. If a handler chooses _not_ to set a result, the next handler in the chain is invoked.

This chain of precedence makes it possible to define a default handler, and then temporarily override that handler. For example, a root level handler may provide default error recovery behavior. But a specific view in the application may know how to recover from a certain error without prompting the user. It could register a handler whilst it's activated, then dispose of that registration when it deactivates. Obviously such an approach requires a shared interaction instance.

> **Note** The `Interaction<TInteractionData>` class is designed to be extensible. Subclasses can change the behavior of `Handle` such that it does not exhibit the behavior described above. For example, you could write an implementation that tries only the first handler in the list.

## Unhandled Interactions

If there are no handlers for a given interaction, or none of the handlers set a result, the interaction is itself considered unhandled. In this circumstance, the invocation of `Handle` will result in an `UnhandledInteractionException<TInteractionData>` being thrown. This exception includes both an `Interaction` and `InteractionData` property, so you can examine the details of the failed interaction.

## Testing

You can easily test interaction logic in view models by registering a handler against the interaction:

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

If your test is hooking into a shared interaction, you probably want to dispose of the registration before your test returns:

```cs
[Fact]
public async Task interaction_test()
{
    var fixture = new SomeViewModel();
    
    using (Interactionss.Error.RegisterHandler(interaction => interaction.SetResult(ErrorRecoveryOption.Abort)))
    {
        fixture.SomeMethodAsync();
        
        // assert abort here
    }
}
```