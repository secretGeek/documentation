# Commands

Commands encapsulate logic to be executed in response to some user action. Examples of actions are clicking a **Save** menu item, tapping a phone icon, or stretching an image. Respectively, the associated logic might be to save outstanding changes, instigate a phone call, or zoom into the image.

Commands may or may not be executable in a given situation. For example, the command backing the **Save** menu item might be unavailable if there are no unsaved changes.

In MVVM, the view model does not concern itself with *how* a user executes a particular command - that decision is left to the view. But typically it's best if the execution logic resides in the view model so that changes to the view model's state can be encapsulated. Commands facilitate this separation between the *how* and the *what*.

If you've done any UI development in .NET, you're likely familiar with the [`ICommand`](https://msdn.microsoft.com/en-us/library/system.windows.input.icommand(v=vs.110).aspx) interface. It is an abstraction that can be used for exactly the purposes described. It's `CanExecute` method can be used to determine whether a command can be executed. If so, one can then invoke the `Execute` method. The `CanExecuteChanged` event can be used by interested parties to know when the command's executability has changed. For example, a `Button` bound to a particular `ICommand` would want to update its enabled state based on the executability of the command.

But the `ICommand` interface isn't an ideal abstraction. It fails to elegantly accommodate long-running commands, such as those that perform I/O. Moreover, its interface is imperative, not reactive. This makes it far less amenable to a reactive code base.

## Reactive Commands

## A Compelling Example

## Asynchronous versus Synchronous Commands

## Command Parameters

## Command Values

## Controlling Executability

## Controlling Scheduling

## Handling Errors

## Binding

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

