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

Interactions can be raised either locally or globally. When raised locally, the handler (usually a view) must have direct access to the interaction instance. When raised globally, the handler need not have direct access.

The advantage of global interactions is that you can handle the same interaction in one location. This is particularly useful for errors because there are usually many sources for errors throughout the application, but typically only one useful recourse for handling them.

The primary disadvantage of global interactions is that seemingly unrelated components can become coupled. If a view model needs an answer to a specific question in a confined context, having the associated view handle the interaction directly is usually a better choice.

Another disadvantage of global interactions is that they can necessitate the creation of application-specific `UserInteraction<TResult>` subclasses. If, for example, a `UserInteraction<bool>` is raised globally to get an answer to two completely different questions, a global handler does not have sufficient information to ask the right question.

For these reasons, local interactions are recommended in those cases where they suffice. 

## An Example

Here is an example of a local interaction:

```cs
public class SomeViewModel : ReactiveObject
{
    private readonly Subject<UserInteraction<bool>> confirmFileDeletion;
    
    public SomeViewModel()
    {
        this.confirmFileDeletion = new Subject<UserInteraction<bool>>();
    }

    public IObservable<UserInteraction<bool>> ConfirmFileDeletion => this.confirmFileDeletion;

    private async Task SomeMethodAsync()
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

public class SomeView
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
public class SomeViewModel : ReactiveObject
{
    private async Task SomeMethodAsync()
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

public class SomeView
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

## Unhandled Interactions

## Testing