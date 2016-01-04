> This section is a work in progress by Ryan Davis and Geoffrey Huntley. Speak with them on Slack if you want to help out.


# WhenAny

# Watching Single Property

```cs
this.WhenAny(x => x.Foo, x => x.Value)
```

* Watch a number of properties:

```cs
// This signals when any one of these three properties change
this.WhenAny(x => x.Red, x => x.Green, x => x.Blue,
    (r,g,b) => new Color(r.Value, g.Value, b.Value));
```

* Watch a nested property

```cs
this.WhenAny(x => x.Foo.Bar.Baz, x => x.Value);
```


# Watching Multiple Properties 

```cs
// This signals when any one of these three properties change
this.WhenAny(x => x.Red, x => x.Green, x => x.Blue,
    (r,g,b) => new Color(r.Value, g.Value, b.Value));
```

* Watch a nested property

```cs
this.WhenAny(x => x.Foo.Bar.Baz, x => x.Value);
```

# Explain Different Types

There are several variants of WhenAny that are useful:

##WhenAny and WhenAnyValue?

* **WhenAnyValue** is a shortcut for the most common usage of **WhenAny** 

The following two statements are equivalent and return an observable that yields the updated value of `SearchText` on every change:

`this.WhenAny(x => x.SearchText, x=> x.Value)`
`this.WhenAnyValue(x => x.SearchText)`

`WhenAnyValue` covers most common scenarios, and results in simpler looking code.

* Working with `WhenAny` directly gives you the opportunity to provide a selector for the output observable. This can be useful in scenarios such as the following:
* You want to transform the property that has been changed, or select a property or constant unrelated to the property that has changed:
* `this.WhenAny(x => x.SearchText, x => x.Length)` returns an observable that yields the length of the query on every change to `SearchText`
* `this.WhenAny(x => x.SearchText, _ => DateTime.Now)` returns an observable that yields the current time on every change to `SearchText`
* You want access the `ObservedChange<,>` object that `WhenAny` produces. This is typically useful for framework code or extension methods. `ObservedChange` exposes the following properties:
* `Value` - the updated value
* `Sender` - the object whose has property changed (i.e. the `this` in `this.WhenAny`, typically the ViewModel)
* `Expression` - the expression that changed on `Sender`

## WhenAnyObservable
// ughhhhhh i will reword it
`WhenAnyObservable` acts a lot like the Rx operator `CombineLatest`, in that it watches multiple observables and yields the latest value of each whenever any update. However, it can be used within the context of ReactiveUI applications to watch properties on an object that may be replaced at some point. 
For example, consider a view that needs to subscribe to a command on its viewmodel. `ViewModel.MyCommand.Subscribe(..)` will work fine for most cases. But what

* **WhenAnyObservable** - You provide a list of properties which are Observables, and they are combined via `CombineLatest`. This is particularly useful for subscribing to Commands in a View:


## Theory
## Best Practices


## How not to hang yourself

### Cold Observable semantics

* `WhenAny` is a purely cold Observable, which eventually directly connects to
   UI component events. For events such as DependencyProperties, this could
   potentially be a (minor) place to optimize, via `Publish`.

* if you chain a `WhenAny` to a `ToProperty`, but don't call `.Value` on it or subscribe to a WhenAny on /that/ property, still nothing will happen
  (but hey, if you're not checking the derived property, it doesn't matter that it didn't change, right? like a tree falling in the woods or something)

### INPC
* (turn this into real words etc)
* Watched properties need to implement ReactiveUI's `RaiseAndSetIfChanged` or standard INPC* (* pretty sure INPC works, need to verify*). if you attempt to whenany on something without either of these a warning will be issued at runtime (ILogger)



## Examples 'Gallery'

Below are a set of typical `WhenAny` usage examples with a brief explanation.

### The 'live query' search text from the main example?
### Validation for a login screen 
### WhenAnyObs from the View
### ...


                
                
## OPAHS, OPAHS, last night I actually dreamnt of OPAHS (what the hell?)
## ??!?!


* `WhenAny` always provides you with the current value as soon as you Subscribe
   to it - it is effectively a BehaviorSubject.


## WhenAnyValue

* **WhenAnyValue** - You provide a single property and you get the latest value.


# Usage Considerations





# The Old Stuff


# Semantics of WhenAny + WhenAnyValue

One of the core features of ReactiveUI is to be able to convert properties to
Observables, via `WhenAny`, and to convert Observables into Properties, via a
method called `ToProperty`.

### What can WhenAny do?

* Watch a single property:

```cs
this.WhenAny(x => x.Foo, x => x.Value)
```

* Watch a number of properties:

```cs
// This signals when any one of these three properties change
this.WhenAny(x => x.Red, x => x.Green, x => x.Blue,
    (r,g,b) => new Color(r.Value, g.Value, b.Value));
```

* Watch a nested property

```cs
this.WhenAny(x => x.Foo.Bar.Baz, x => x.Value);
```

### Different Kinds of WhenAnys

WhenAny has a few specific properties that you should know about:

### Expression Evaluation Semantics

Consider the following code:

```cs
this.WhenAny(x => x.Foo.Bar.Baz, _ => "Hello!")
    .Subscribe(x => Console.WriteLine(x));

// Example 1
this.Foo.Bar.Baz = null;
>>> Hello!

// Example 2: Nothing printed!
this.Foo.Bar = null;

// Example 3
this.Foo.Bar = new Bar() { Baz = "Something" };
>>> Hello!
```

`WhenAny` will only send notifications if reading the given Expression would not
throw a Null reference exception. In Example 1, even though Baz is `null`,
because the expression could be evaluated, you get a notification.

In Example 2 however, evaluating `this.Foo.Bar.Baz` wouldn't give you `null`, it
would crash. `WhenAny` therefore suppresses any notifications from being
generated. Setting `Bar` to a new value generates a new notification.

### Distinctness

`WhenAny` only tells you when the *final value* of the expression has
**changed**. This is true even if the resulting change is because of an
intermediate value in the expression chain. Here's an explaining example:

```cs
this.WhenAny(x => x.Foo.Bar.Baz, _ => "Hello!")
    .Subscribe(x => Console.WriteLine(x));

// Example 1
this.Foo.Bar.Baz = "Something";
>>> Hello!

// Example 2: Nothing printed!
this.Foo.Bar.Baz = "Something";

// Example 3: Still nothing
this.Foo.Bar = new Bar() { Baz = "Something" };

// Example 4: The result changes, so we print
this.Foo.Bar = new Bar() { Baz = "Else" };
>>> Hello!
```

### More things



# ToC



# WhenAny Notes

** overview of whenany **

 -> exposing property changes as signal/observable; now we can combine, transform, aggregate declaratively etc.
 -> typical uses - perform actions in response to property changes, create 'derived' properties based on the values of one or many other properties, implement validation logic
 -> can watch one or multiple properties

// probably some examples go here

** things that need to be covered **:

        whenany vs whenanyvalue:
         -> most application code will not need to deal with the `ObservedChange` returned by whenany and can just use whenanyvalue 
         -> it's worth mentioning as every user will see `WhenAny` alongside `WhenAnyValue` when using intellisense
        
        whenany + toproperty pattern:
         -> whenany turns properties into observables        
         -> toproperty turns observables into readonly/derived properties
         -> ?? 
         -> the pattern is cool for defining 'readonly' / 'derived' values based on other properties; updated when the original properties are changed
         -> can link to the real toProperty/OAPH topic for more detail

        whenanyobservable for safe whenany from outside of the vm:
         -> eg view watching a viewmodel property 
                
        how to not get 🔥'd:


         -> nothing happens until you subscribe
                if you try to test a whenany using something like a Do statement without a subscription at the end of the chain, you're gonna have a Bad Time. 
                if you chain a `WhenAny` to a `ToProperty`, but don't call `.Value` on it or subscribe to a WhenAny on /that/ property, still nothing will happen
                (but hey, if you're not checking the derived property, it doesn't matter that it didn't change, right? like a tree falling in the woods or something)

         -> subscribers to a whenany will be given the latest value immediately

         -> ?? there are possibly some scenarios where you need to publish/refcount in order to use a whenany (need to look at a past proj to find/verify this, I could just be making it up)

** ideas for examples **



 -> considering the 'live search' example, the changing searchtext is a good candidate 
    - demonstrates Good Things that you get thanks to WhenAny like ability to filter invalid, transform(e.g. .trim()), distinctuntilchanged, throttle etc.

 -> a composite whenany example could be used to demonstrate specifying a canExecute criteria.. 
    - login screen is simple and familiar but we can conjure up reasonably complex validation criteria (user + pass are entered, length is within ranges, appropriate characters used, etc) to demo functionality .