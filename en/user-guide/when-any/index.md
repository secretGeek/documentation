> **Warning** This section is a work in progress by Ryan Davis and Geoffrey Huntley. Speak with them on Slack if you want to help out.

# Introduction

One of the core features of ReactiveUI is to be able to represent Properties as
Observables, and conversely, represent Observables as Properties. 
Treating changes to a property is an Observable stream is fundamental to FRP application design and
the `WhenAny` operators are ReactiveUI's construct for achieving this.

## Why model changes as an observable?

> **Warning** Wordsmithing required / incomplete

- we can define our app in terms of changes in properties
- we can declaratively compose logic over these changes using rx operators
- treating asynchrosity and multiple values as first class citizens allows us to easily express logic that would be very difficult/error prone in an imperative manner
- etc.

## What does it look like?

###Watching single property

This returns an observable that yields the current value of Foo each time it changes:

```cs
this.WhenAnyValue(x => x.Foo)
```

###Watching a number of properties

This returns an observable that yields a new `Color` with the latest RGB values each time any of the properties change:

```cs
// This signals when any one of these three properties change
this.WhenAnyValue(x => x.Red, x => x.Green, x => x.Blue, 
                 (r,g,b) => new Color(r, g, b));
```

###Watching a nested property

`WhenAny` and friends can look into nested properties and watch for changes, too!

```cs
this.WhenAnyValue(x => x.Foo.Bar.Baz);
```

Cool! But why `WhenAnyValue`? 

#Meet the `WhenAny`s
 
There are a few variants of `WhenAny`, each suited for different scenarios:

##WhenAny and WhenAnyValue

* **WhenAnyValue** is covers the most common usage of **WhenAny**, and is a useful shortcut in many cases.

The following two statements are equivalent and return an observable that yields the updated value of `SearchText` on every change:

`this.WhenAny(x => x.SearchText, x=> x.Value)`
`this.WhenAnyValue(x => x.SearchText)`

If you just need to watch one or more properties for changes, `WhenAnyValue` is quick to type and results in simpler looking code.

* Working with `WhenAny` directly gives you access to the `ObservedChange<,>` object that ReactiveUI produces on each property change. This is typically useful for framework code or extension methods. `ObservedChange` exposes the following properties:
* `Value` - the updated value
* `Sender` - the object whose has property changed (i.e. the `this` in `this.WhenAny`, typically the ViewModel)
* `Expression` - the expression that changed on `Sender`

##WhenAnyObservable

> **Warning** Wordsmithing required / incomplete

`WhenAnyObservable` acts a lot like the Rx operator `CombineLatest`, in that it watches one or multiple observables and allows you to define a projection based on the latest value from each. `WhenAnyObservable` differs from `CombineLatest` in that its parameters are expressions and are not tied to the specific observables present at the time of subscription. This can be handy when wishing to subscribe to a ViewModel command from a View. 

Consider the statement `this.WhenAnyObservable(x => x.ViewModel.DoStuff)` - as opposed to a direct subscription to the `ViewModel.DoStuff` command, a subscription to the `WhenAnyObservable` result will continue to signal on new command invocations if the ViewModel is replaced at a later point in time. Without `WhenAnyObservable`, the View would need create another direct subscription to the new ViewModel's `DoStuff` in order to receive signals from it.

# 'Derived' values using `ToProperty`

(only a brief overview here, as `ToProperty`/`ObservableAsPropertyHelper` has its own page)

# Best Practices / How not to hang yourself

> **Warning** Wordsmithing required / incomplete

There are a few things to keep in mind when when using `WhenAny` and friends:

### INPC

* Watched properties need to implement ReactiveUI's `RaiseAndSetIfChanged` or standard INPC* (* pretty sure INPC works, need to verify*). if you attempt to whenany on something without either of these a warning will be issued at runtime (ILogger)

### Cold Observable semantics

* `WhenAny` is a purely cold Observable, which eventually directly connects to
   UI component events. For events such as DependencyProperties, this could
   potentially be a (minor) place to optimize, via `Publish`.

* if you chain a `WhenAny` to a `ToProperty`, but don't call `.Value` on it or subscribe to a WhenAny on /that/ property, still nothing will happen
  (but hey, if you're not checking the derived property, it doesn't matter that it didn't change, right? like a tree falling in the woods or something)

### Behavioural Semantics 
* `WhenAny` always provides you with the current value as soon as you Subscribe
   to it - it is effectively a BehaviorSubject.

# Examples 'Gallery'

Below are a set of typical `WhenAny` usage examples with a brief explanation.

* The 'live query' search text from the main example?
* Validation for a login screen 
* WhenAnyObs from the View
* ..
