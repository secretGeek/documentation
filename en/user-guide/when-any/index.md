> **Warning** This section is a work in progress by Ryan Davis and Geoffrey Huntley. Speak with them on Slack if you want to help out.

One of the core features of ReactiveUI is to be able to represent Properties as
Observables - and conversely - Observables as Properties. In particular, the idea that changes to a property can be represented as an Observable stream of events is fundamental the effective design of reactive applications.

The motivation is intuitive enough when you think about it. It's not hard to imagine that changes to a property can be considered events - that's how `INotifyPropertyChanged` works. From there, the same argument for using Rx over events applies. In the context of MVVM application design specifically, modelling property changes as observables leads to several advantages:

- The logic of an application can be defined in terms of changes to properties
- This logic can be composed and expressed declaratively, using the power of Rx operators
- Concepts like time and asynchronicity become easier to reason about, due to their first-class treatment in an Observable context.

ReactiveUI provides several variants of `WhenAny` to help you work with properties as an observable stream. 

## Basic syntax

The following examples demonstrate simple uses of `WhenAnyValue`, the `WhenAny` variant you are likely to use most frequently.

####Watching single property

This returns an observable that yields the current value of Foo each time it changes:

```cs
this.WhenAnyValue(x => x.Foo)
```

####Watching a number of properties

This returns an observable that yields a new `Color` with the latest RGB values each time any of the properties change. The final parameter is a selector describing how to combine the three observed properties:

```cs
this.WhenAnyValue(x => x.Red, x => x.Green, x => x.Blue, 
                 (r,g,b) => new Color(r, g, b));
```

####Watching a nested property

`WhenAny` variants can observe nested properties for changes, too:

```cs
this.WhenAnyValue(x => x.Foo.Bar.Baz);
```

## Idiomatic usage

Naturally, once you have an observable you can `Subscribe` to it to perform actions response to the changed values. However, given the design of ReactiveUI is centered around the observable, in many cases there will be a Better Way to achieve what you want. Below are some typical usages of the observables returned by  `WhenAny` variants:

#### Exposing 'calculated' properties

In general, using `Subscribe` on a `WhenAny` observable (or any observable, for that matter) just to set a property is likely a code smell. Idiomatically, the `ToProperty` operator is used to create a 'read-only' calculated property that can be exposed to the rest of your application, but not set in any manner but the `WhenAny` chain that preceded it:

```cs
this.WhenAnyValue(x => x.SearchText, x => x.Length)
    .DistinctUntilChanged()
    .ToProperty(this, x => x.SearchTextLength, out _searchTextLength);
```

This initialises the `SearchTextLength` property (an [ObservableAsPropertyHelper](../observableaspropertyhelper/index.md) property) as a property that will be updated with the current search text length every time it changes. The property cannot be set in any other manner and raises change notifications so can itself be used in a `WhenAny` expression or a binding. 

See the [ObservableAsPropertyHelper](../observableaspropertyhelper/index.md) section for more information on this pattern.

#### Supporting validation as a `CanExecute` criteria

...

#### Performing view-specific transforms as an an input to `BindTo`

... 


##`WhenAny` Variants
 
Several variants of `WhenAny` exist, suited for different scenarios.

####WhenAny vs WhenAnyValue

* **WhenAnyValue** is covers the most common usage of **WhenAny**, and is a useful shortcut in many cases.

The following two statements are equivalent and return an observable that yields the updated value of `SearchText` on every change:

- `this.WhenAny(x => x.SearchText, x=> x.Value)`
- `this.WhenAnyValue(x => x.SearchText)`

When needing to observe one or many properties for changes, `WhenAnyValue` is quick to type and results in simpler looking code. Working with `WhenAny` directly gives you access to the `ObservedChange<,>` object that ReactiveUI produces on each property change. This is typically useful for framework code or extension methods. `ObservedChange` exposes the following properties:
* `Value` - the updated value
* `Sender` - the object whose has property changed 
* `Expression` - the expression that changed

At the risk of extreme repetition - use `WhenAnyValue` unless you know you need `WhenAny`. 

####WhenAnyObservable

> **Warning** Wordsmithing required / incomplete

`WhenAnyObservable` acts a lot like the Rx operator `CombineLatest`, in that it watches one or multiple observables and allows you to define a projection based on the latest value from each. `WhenAnyObservable` differs from `CombineLatest` in that its parameters are expressions and are not tied to the specific observables present at the time of subscription. This can be handy when wishing to subscribe to a ViewModel command from a View. 

Consider the statement `this.WhenAnyObservable(x => x.ViewModel.DoStuff)` - as opposed to a direct subscription to the `ViewModel.DoStuff` command, a subscription to the `WhenAnyObservable` result will continue to signal on new command invocations if the ViewModel is replaced at a later point in time. Without `WhenAnyObservable`, the View would need create another direct subscription to the new ViewModel's `DoStuff` in order to receive signals from it.

## Best Practices / How not to hang yourself

> **Warning** Wordsmithing required / incomplete

There are a few things to keep in mind when when using `WhenAny` and friends:

#### INPC

* Watched properties need to implement ReactiveUI's `RaiseAndSetIfChanged` or standard INPC* (* pretty sure INPC works, need to verify*). if you attempt to whenany on something without either of these a warning will be issued at runtime (ILogger)

#### Cold Observable semantics

* `WhenAny` is a purely cold Observable, which eventually directly connects to
   UI component events. For events such as DependencyProperties, this could
   potentially be a (minor) place to optimize, via `Publish`.

* if you chain a `WhenAny` to a `ToProperty`, but don't call `.Value` on it or subscribe to a WhenAny on /that/ property, still nothing will happen
  (but hey, if you're not checking the derived property, it doesn't matter that it didn't change, right? like a tree falling in the woods or something)

#### Behavioural Semantics 
* `WhenAny` always provides you with the current value as soon as you Subscribe
   to it - it is effectively a BehaviorSubject.

# Examples 'Gallery'

Below are a set of typical `WhenAny` usage examples with a brief explanation.

* The 'live query' search text from the main example?
* Validation for a login screen 
* WhenAnyObs from the View
* ..
