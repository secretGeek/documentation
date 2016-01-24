# User Interactions

At times you may find yourself writing view model code that needs to confirm something with the user. For example, checking if it's OK to delete a file, or asking what to do about an error that has occurred.

It might be tempting to simply throw up a message box right from within the view model. But that would be a mistake. Not only does this tie your view model to a particular UI technology, it also makes testing difficult (or even impossible).

Instead what is needed is a means of suspending the view model's execution path until some data is provided by the user. ReactiveUI's user interaction mechanism facilitates just this.

## API Overview

Underpinning the user interaction infrastructure is the `UserInteraction` class. This abstract class represents the interaction. That is, it models the process of asking the user for data, and includes the user's response.

The generic `UserInteraction<TResult>` class extends `UserInteraction` and strongly-types the result of the interaction. For example, a `UserInteraction<bool>` is capable of obtaining a yes/no answer from the user.

For richer interactions that require more than a typed result, you can subclass `UserInteraction<TResult>`. In fact, ReactiveUI provides one such subclass called `UserErrorInteraction<TResult>`. This class adds an `Error` property (of type `Exception`), which makes it a suitable option in error recovery scenarios.

Interactions are usually created by your view model. They are then "raised" either locally or globally to obtain a result from outside the view model.

## Local versus Global Interactions

Interactions can be raised either locally or globally. If raised locally, the handler (usually a view) must have direct access to the interaction instance. If raised globally, the handler need not have direct access.

The advantage of global interactions is that you can handle the same interaction in one location. This is particularly useful for errors because there are usually many sources for errors throughout the application, but typically only one useful recourse for handling them.

The primary disadvantage of global interactions is that seemingly unrelated components can become coupled. If a view model needs an answer to a specific question in a confined context, having the associated view handle the interaction directly is usually a better choice.

Another disadvantage of global interactions is that they can necessitate the creation of application-specific `UserInteraction<TResult>` subclasses. If, for example, two different view models raise a `UserInteraction<bool>` globally to get an answer to two completely different questions, a global handler does not have sufficient information to distinguish between them.

For these reasons, local interactions are recommended in those cases where they suffice. 

## An Example

Here is an example of a local interaction:

```cs
public class LocalInteractionViewModel : ReactiveObject
{
    private readonly Subject<UserInteraction<bool>> confirmFileDeletion;
    
    public LocalInteractionViewModel()
    {
        this.confirmFileDeletion = new Subject<UserInteraction<bool>>();
    }

    public IObservable<UserInteraction<bool>> ConfirmFileDeletion => this.confirmFileDeletion;

    public async Task DeleteFileAsync()
    {
        // create the interaction instance
        var confirmFileDeletion = new UserInteraction<bool>();
    
        // tick it through the observable so the view gets it
        this.confirmFileDeletion.OnNext(confirmFileDeletion);
    
        // raise it and await the result
        var result = await confirmFileDeletion.Raise();
    
        if (result)
        {
            // delete the file
        }
    }
}

public class LocalInteractionView
{
    public SomeView()
    {
        this
            .WhenAnyObservable(x => x.ViewModel.ConfirmFileDeletion)
            .Subscribe(
                async interaction =>
                {
                    var deleteIt = await this.DisplayAlert(
                        "Confirm Delete",
                        "Are you sure you want to delete this super important file?",
                        "YES",
                        "NO");
                
                    interaction.SetResult(deleteIt);
                });
            
    }
}
```

And here is the same use case implemented as a global interaction:

```cs
public class GlobalInteractionViewModel : ReactiveObject
{
    public async Task DeleteFileAsync()
    {
        // create the interaction instance
        var confirmFileDeletion = new UserInteraction<bool>();

        // raise it globally and await the result
        var result = await confirmFileDeletion.RaiseGlobal();
    
        if (result)
        {
            // delete the file
        }
    }
}

public class GlobalInteractionView
{
    public SomeView()
    {
        UserInteraction
            .RegisterGlobalHandler<UserInteraction<bool>>(
                async interaction =>
                {
                    var deleteIt = await this.DisplayAlert(
                        "Confirm Delete",
                        "Are you sure you want to delete this super important file?",
                        "YES",
                        "NO");
                
                    interaction.SetResult(deleteIt);
                });
    }
}
```

> **Note** For the sake of clarity, the example code here mixes TPL and Rx code. Production code would normally stick with one or the other.

## Handler Precedence

Global handlers can form a chain. Any number of handlers can be registered, and later registrations are deemed of higher priority than earlier registrations. When an interaction is raised globally, each global handler is given the _opportunity_ to handle that interaction (i.e. set a result). The handler is under no obligation to actually handle the interaction. If a handler chooses not to set a result, the next handler in the chain is invoked.

This chain of precedence makes it possible to define default handlers at a root level of your application, then override those handlers from higher level components. For example, a root level handler may provide default error recovery behavior. But a specific view in the application may know how to recover from a certain error without prompting the user. It could register a global handler whilst it's activated, then dispose of that registration when it deactivates.

> **Note** Handler precedence is only relevant to global interactions. Generally speaking, local interactions will normally only have one handler.

## Unhandled Interactions

Any interaction, be it global or local, can go unhandled. If there are no handlers, or none of the handlers set a result, the interaction is unhandled. This is always a programming error.

For local interactions, this is likely because you've forgotten to hook into the interaction from your view. Or perhaps you've forgotten to call `SetResult` against the interaction instance. Either way, the end result is that your view model will stall when it awaits the interaction result. Because nothing has handled that interaction, its result is never set and so the view model waits indefinitely.

For global interactions, the handler chain mechanism gives each registered handler an opportunity to handle the interaction. If the end of the chain is reached and no handler has handled the interaction, an `UnhandledUserInteractionException` is thrown.

## Testing

Regardless of whether you're using global or local interactions, you can easily test user interaction logic in view models.

Local interactions need to hook into the interaction through whatever means the view model provides:

```cs
[Fact]
public async Task local_interaction_test()
{
    var fixture = new SomeViewModel();
    fixture
        .WhenAnyObservable(x => x.ConfirmFileDeletion)
        .Subscribe(i => i.SetResult(true));
    
    await fixture.DeleteFileAsync();
    
    Assert.True(/* file was deleted */);
}
```

For global interactions, your unit test can easily register a temporary handler:

```cs
[Fact]
public async Task global_interaction_test()
{
    var fixture = new SomeViewModel();
    
    using (UserInteraction.RegisterGlobalHandler<UserInteraction<bool>>(i => i.SetResult(true))
    {
        await fixture.DeleteFileAsync();
        
        Assert.True(/* file was deleted */);
    }
}
```