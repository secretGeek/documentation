# Version 7.0

## ReactiveCommand is Better

`ReactiveCommand` is completely rewritten again (sorry).

* interfaces are gone. Any use of `IReactiveCommand` should be replaced with `ReactiveCommand`, possibly with type information (see below).
* static creation methods have changed:
    * execution logic is _always_ required when calling `CreateXxx` methods, including synchronous commands (i.e. those created with `Create`). So rather than calling `Create` and then subscribing, you call `Create` and pass in your execution logic right then and there.
    * for consistency, the execution logic is always provided as the first parameter. Other parameters (`canExecute`, `scheduler`) are optional.
    * `CreateAsyncObservable` is now called `CreateFromObservable`.
    * `CreateAsyncTask` is now called `CreateFromTask`.
* parameter types are formalized by `TParam` in `ReactiveCommand<TParam, TResult>`.
    * if your command takes a parameter, you no longer take an `object` and cast it. Instead, you explicitly specify the parameter type when creating the command (of course, you can still choose `object` as your `TParam` if that makes sense, perhaps as an intermediary migration step).
* `ICommand` is now implemented explicitly. As a result:
    * the `Execute` exposed by `ReactiveCommand` is reactive (it returns `IObservable<TResult>`). It is therefore lazy and won't do anything unless something subscribes to it.
    * `CanExecuteObservable` is now called `CanExecute`.
* observables such as `CanExecute` and `IsExecuting` are now behavioral. That is, they will always provide the current value to new subscribers.
* `RoutingState` has been updated to use the new implementation. Consequently, any use of its commands will be affected per the above.
* the `ToCommand` extension method has been removed. This was a simple convenience to take an `IObservable<bool>` and use it as the `canExecute` pipeline for a new command. If you're using `ToCommand`, you can just replace it with a call to one of the creation methods on `ReactiveCommand`.

**Old:**

```cs
var canExecute = ...;
var someCommand = ReactiveCommand.Create(canExecute);
someCommand.Subscribe(x => /* execution logic */);

var someAsyncCommand1 = ReactiveCommand.CreateAsyncObservable(canExecute, someObservableMethod);
var someAsyncCommand2 = ReactiveCommand.CreateAsyncTask(canExecute, someTaskMethod);

someCommand.Execute();
```

**New:**

```cs
var canExecute = ...;
var someCommand = ReactiveCommand.Create(() => /* execution logic */);

var someAsyncCommand1 = ReactiveCommand.CreateFromObservable(someObservableMethod, canExecute);
var someAsyncCommand2 = ReactiveCommand.CreateFromTask(someTaskMethod, canExecute);

someCommand.Execute().Subscribe();
```

For more details, please see the [extensive documentation](https://docs.reactiveui.net/en/user-guide/commands/index.html) on this topic.

> **Note** To enable you to ease into the migration, all previous types are available under the `ReactiveUI.Legacy` namespace. Note, however, that there is no legacy version of `RoutingState`, so any code you have that interacts with its command may require minor updates.

## Interactions are New and Exciting

`UserError` has been generalized and re-imagined. We call it interactions, and we think you'll like it. We did this in part because people were feeling icky using `UserError` for non-error scenarios. Basically, we realized that people need a general mechanism via which a view model can ask a question, and wait for the answer. It doesn't have to be an error - we're not that pessimistic! You could be asking to confirm a file deletion, or maybe how the weather is out there in the analog world.

Migrating from `UserError` to the interactions infrastructure is not really a case of one-for-one substitution. But here are some tips to get you started:

* read through [the documentation](http://docs.reactiveui.net/en/user-guide/interactions/index.html) first.
* decide whether you need shared interactions and, if so, define them in an appropriate place for your application (often just a static class).
* for any non-shared interactions, have your view model create an instance of the interaction and expose it via a property.
* typically you want the corresponding view to handle interactions by calling one of the `RegisterHandler` methods on the interaction exposed by the view model.
* the view model can call `Handle` on the interaction, passing in an input value.
* Recovery commands are no longer a built-in thing. If you need such a mechanism for your interactions, you are encouraged to write an appropriate class and use it as the input for your interaction.

> **Note** To enable you to ease into the migration, all previous types are available under the `ReactiveUI.Legacy` namespace.

## Platform-independent

* `ReactiveCommand` re-written to improve behavior and APIs [#954](https://github.com/reactiveui/ReactiveUI/issues/954) [#955](https://github.com/reactiveui/ReactiveUI/issues/955) [#727](https://github.com/reactiveui/ReactiveUI/issues/727)
* `UserError` deprecated and replaced with the more flexible `Interaction<TInput, TOutput>` [#920](https://github.com/reactiveui/ReactiveUI/issues/920)
* removed fallback value support from bindings [#848](https://github.com/reactiveui/ReactiveUI/issues/848)
* removed binding overloads with implicit view resolution [#846](https://github.com/reactiveui/ReactiveUI/issues/846)
* fixed `WhenAnyObservable` bug that could result in a `NullReferenceException` [#830](https://github.com/reactiveui/ReactiveUI/issues/830)
* `ToProperty` now provides the initial value immediately [#815](https://github.com/reactiveui/ReactiveUI/issues/815)
* `CreateCollection` now takes an optional scheduler [#1009](https://github.com/reactiveui/ReactiveUI/issues/1009)

## Xamarin Forms

* `ViewModelViewHost` now extends `ContentView`, not `StackLayout` [#947](https://github.com/reactiveui/ReactiveUI/issues/947)
* fixed `RoutedViewHost` bugs [#890](https://github.com/reactiveui/ReactiveUI/issues/890)
* fixed `ViewModelViewHost` bugs [#946](https://github.com/reactiveui/ReactiveUI/issues/946) [#891](https://github.com/reactiveui/ReactiveUI/issues/891)
* added `ReactiveTextCell`, `ReactiveEntryCell`, `ReactiveSwitchCell`, `ReactiveImageCell`, and `ReactiveViewCell` [#882](https://github.com/reactiveui/ReactiveUI/issues/882)
* fixed events NuGet package to support Xamarin Forms [#881](https://github.com/reactiveui/ReactiveUI/issues/881)
* enhanced activation support [#949](https://github.com/reactiveui/ReactiveUI/issues/949)

## iOS

* added `ReactiveSplitViewController` [#899](https://github.com/reactiveui/ReactiveUI/issues/899)
* `ViewModelViewHost` rewritten [#868](https://github.com/reactiveui/ReactiveUI/issues/868)
* rewrote `ReactiveTableViewSource` and supporting infrastructure to fix several deficiencies [#740](https://github.com/reactiveui/ReactiveUI/issues/740) [#808](https://github.com/reactiveui/ReactiveUI/issues/808) [#867](https://github.com/reactiveui/ReactiveUI/issues/867)

## Android
* Min target is now MonoAndroid403 aka Android 4.0.3 / API Level 15

## XAML

* `ViewModelViewHost` now includes a scalar `ViewContract` property [#870](https://github.com/reactiveui/ReactiveUI/issues/870)
* include `XmlnsDefinition` for supported platforms [#863](https://github.com/reactiveui/ReactiveUI/issues/863)