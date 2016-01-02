# Dependency Injection

Dependency resolution is very useful for moving logic that would normally have to be in platform-specific code, into the shared platform code. First, we need to define an Interface for something that we want to use - this example isn't a Best Practice, but it's illustrative.


Dependency resolution is a feature built into the core framework, which allows libraries and ReactiveUI itself to use classes that are in other libraries without taking a direct reference to them. This is quite useful for cross-platform applications, as it allows portable code to use non-portable APIs, as long as they can be described via an Interface.

ReactiveUI's use of dependency resolution can more properly be called the Service Location pattern. Put thought into how you use this API, as it can either be used effectively to make code more testable, or when used poorly, makes code more difficult to test and understand, as the [Resolver itself can effectively become part of the class's state](http://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/), but in an implicit and non-obvious way.

Since ReactiveUI 6, [Splat](https://github.com/paulcbetts/splat) is used by ReactiveUI for service location and dependency injection. Earlier versions included a RxUI resolver. If you come across samples for RxUI versions earlier than 6, you should replace references to `RxApp.DependencyResolver` with `Locator.Current` and `RxApp.MutableResolver` with `Locator.CurrentMutable`.

## Resolution

Splat provides methods to resolve dependencies to single or multiple instances. 
 
```csharp
var toaster = Locator.Current.GetService<IToaster>();
var allToasterImpls = Locator.Current.GetServices<IToaster>();
```

## Registration

Splat supports on-demand new'ing, constant and lazy registration of dependencies. 

```cs
// Create a new Toaster any time someone asks
Locator.CurrentMutable.Register(() => new Toaster(), typeof(IToaster));

// Register a singleton instance
Locator.CurrentMutable.RegisterConstant(new ExtraGoodToaster(), typeof(IToaster));

// Register a singleton which won't get created until the first user accesses it
Locator.CurrentMutable.RegisterLazySingleton(() => new LazyToaster(), typeof(IToaster));
```

## Advanced
Splat's dependency resolver, accessible using `Locator.Current` conceptually resembles the below:

```csharp
public interface IDependencyResolver
{
    // Returns the most recent service registered to this type and contract
    T GetService<T>(string contract = null);

    // Returns all of the services registerd to this type and contract
    IEnumerable<T> GetServices<T>(string contract = null)
}
```

Given a type T (usually an interface), you can now recieve an implementation of T. If the T registered is very common ("string" for example), or you want to distinguish by a method other than type, you can use the "contract"
parameter which is an arbitrary key that you provide.

The current resolver that ReactiveUI itself will use (as well as what your app
should use as well), is provided by [Splat.ModernDependencyResolver](https://github.com/paulcbetts/splat/blob/b833718d1b7940d1d02403e86864d03d2af5cea7/Splat/ServiceLocation.cs).

