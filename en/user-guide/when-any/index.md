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

##Idiomatic usage

Naturally, once you have an observable you can `Subscribe` to it to perform actions response to the changed values. However, given the design of ReactiveUI is centered around the observable, in many cases there will be a Better Way to achieve what you want. Below are some typical usages of the observables returned by  `WhenAny` variants:

####Exposing 'calculated' properties

In general, using `Subscribe` on a `WhenAny` observable (or any observable, for that matter) just to set a property is likely a code smell. Idiomatically, the `ToProperty` operator is used to create a 'read-only' calculated property that can be exposed to the rest of your application, but not set in any manner but the `WhenAny` chain that preceded it:

```cs
this.WhenAnyValue(x => x.SearchText, x => x.Length)
    .DistinctUntilChanged()
    .ToProperty(this, x => x.SearchTextLength, out _searchTextLength);
```

This initialises the `SearchTextLength` property (an [ObservableAsPropertyHelper](../observableaspropertyhelper/index.md) property) as a property that will be updated with the current search text length every time it changes. The property cannot be set in any other manner and raises change notifications so can itself be used in a `WhenAny` expression or a binding. 

See the [ObservableAsPropertyHelper](../observableaspropertyhelper/index.md) section for more information on this pattern.

####Supporting validation as a `CanExecute` criteria


``cs
this.WhenAny(x => x.UserName, x => x.Password, 
            (user, pass) => !String.IsNullOrWhitespace(

``
...

####Performing view-specific transforms as an an input to `BindTo`

... 


##`WhenAny` Variants
 
Several variants of `WhenAny` exist, suited for different scenarios.

####WhenAny vs WhenAnyValue

* **WhenAnyValue** is covers the most common usage of **WhenAny**, and is a useful shortcut in many cases.

The following two statements are equivalent and return an observable that yields the updated value of `SearchText` on every change:

- `this.WhenAny(x => x.SearchText, x => x.Value)`
- `this.WhenAnyValue(x => x.SearchText)`

When needing to observe one or many properties for changes, `WhenAnyValue` is quick to type and results in simpler looking code. Working with `WhenAny` directly gives you access to the `ObservedChange<,>` object that ReactiveUI produces on each property change. This is typically useful for framework code or extension methods. `ObservedChange` exposes the following properties:
* `Value` - the updated value
* `Sender` - the object whose has property changed 
* `Expression` - the expression that changed.

At the risk of extreme repetition - use `WhenAnyValue` unless you know you need `WhenAny`. 

####WhenAnyObservable

`WhenAnyObservable` acts a lot like the Rx operator `CombineLatest`, in that it watches one or multiple observables and allows you to define a projection based on the latest value from each. `WhenAnyObservable` differs from `CombineLatest` in that its parameters are expressions, rather than direct references to the target observables. The impact of this difference is that the watch set up by `WhenAnyObservable` is not tied to the specific observable instances present at the time of subscription. That is, the observable pointed to by the expression can be replaced later, and the results of the new observable will still be captured. 

An example of where this can come in handy is when a view wants to observe an observable on a viewmodel, but the viewmodel can be replaced during the view's lifetime. Rather than needing to resubscribe to the target observable after every change of viewmodel, you can use `WhenAnyObservable` to specify the 'path' to which watch. This allows you to use a single subscription in the view, regardless of the life of the target viewmodel. 

##Getting the most out of your new `WhenAny`

Using `WhenAny` variants is fairly straightforward. However, there are a few aspects of their behaviour that are worth highlighting.

####INotifyPropertyChanged is required

* Watched properties must implement ReactiveUI's `RaiseAndSetIfChanged` or raise the standard `INotifyPropertyChanged` events. If you attempt to use `WhenAny` on a property without either of these in place, `WhenAny` will produce the current value of the property upon subscription, and nothing thereafter. Additionally, a warning will be issued at run time (ensure you have registered a service for `ILogger` to see this).

####`WhenAny` has cold observable semantics

* `WhenAny` is a purely cold Observable, which eventually directly connects to
   UI component events. For events such as DependencyProperties, this could
   potentially be a (minor) place to optimize, via `Publish`.

* As `ToProperty` is also cold, if a `WhenAny` is chained to a `ToProperty`, the target `ObservableAsPropertyHelper` must be checked (`.Value`) or observed (e.g. used in a binding or used as part of another `WhenAny` with a subscription) for any of the chain to execute set up. 

####`WhenAny` has behavioural observable semantics 
* `WhenAny` always provides you with the current value as soon as you Subscribe to it - it is effectively a BehaviorSubject.

#Relevant Samples

Samples demonstrating `WhenAny` use will be listed below.
