# Reactive Extensions

One of the most confusing aspects of the Reactive Extensions is that of ["hot", "cold", and "warm" observables](http://www.introtorx.com/content/v1.0.10621.0/14_HotAndColdObservables.html) (event streams). In short, given just a method or function declaration like this:

	IObservable<string> Search(string query)

It is impossible to tell whether subscribing to (observing) that `IObservable` will involve side effects. If it does involve side effects, it’s also impossible to tell whether each subscription has a side effect, or if only the first one does. Whilst this example is contrived, it demonstrates a real, pervasive problem that makes it harder  at first for new comers to understand Rx code at first glance. 

{% video %}https://www.youtube.com/watch?v=IDy21J75eyU{% endvideo %}

As such we also recommend [watching this video](https://www.youtube.com/watch?v=IDy21J75eyU), reading [this article](http://www.introtorx.com/content/v1.0.10621.0/14_HotAndColdObservables.html) and [playing with the marbles](http://rxmarbles.com/) to familiarize yourself with the fundamentals.
