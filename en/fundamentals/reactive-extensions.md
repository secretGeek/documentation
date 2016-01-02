# Reactive Extensions

The Reactive Extensions are a library for composing asynchronous and event-based programs using observable collections. 


## WIP

One of the most confusing aspects of the Reactive Extensions is that of ["hot", "cold", and "warm" observables](http://www.introtorx.com/content/v1.0.10621.0/14_HotAndColdObservables.html) (event streams). In short, given just a method or function declaration like this:

	IObservable<string> Search(string query)

It is impossible to tell whether subscribing to (observing) that `IObservable` will involve side effects. If it does involve side effects, it’s also impossible to tell whether each subscription has a side effect, or if only the first one does. Whilst this example is contrived, it demonstrates a real, pervasive problem that makes it harder  at first for new comers to understand Rx code at first glance. 

As such we also recommend [watching this video](https://www.youtube.com/watch?v=IDy21J75eyU), reading [this article](http://www.introtorx.com/content/v1.0.10621.0/14_HotAndColdObservables.html) and [playing with the marbles](http://rxmarbles.com/) to familiarize yourself with the fundamentals.

## Learning resources

* [Becoming a C# Time Lord](http://channel9.msdn.com/Events/TechEd/Australia/2013/DEV422)

* [101 Rx Samples](http://rxwiki.wikidot.com/101samples)

* [Programming Reactive Extensions and LINQ](http://www.apress.com/programming-reactive-extensions-and-linq?gtmf=s)

* [Reactive Extensions and Your Mouse as Database](http://redsunsoft.com/2015/01/reactive-extensions-rx-mouse-database/)
