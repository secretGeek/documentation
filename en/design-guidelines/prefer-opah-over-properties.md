### Prefer Observable Property Helpers to setting properties explicitly

When a property's value depends on another property, a set of properties, or an 
observable stream, rather than set the value explicitly, use 
`ObservableAsPropertyHelper` with `WhenAny` wherever possible.

__Do__

```csharp
public class RepositoryViewModel : ReactiveObject
{
  public RepositoryViewModel()
  {
    canDoIt = this.WhenAny(x => x.StuffFetched, y => y.OtherStuffNotBusy, (x, y) => x && y)
      .ToProperty(this, x => x.CanDoIt);
  }

  readonly ObservableAsPropertyHelper<bool> canDoIt;
  public bool CanDoIt
  {
    get { return canDoIt.Value; }  
  }	
}
```

__Don't__

```csharp
this.WhenAny(x => x.StuffFetched, y => y.OtherStuffNotBusy, (x, y) => x && y)
  .ObserveOn(RxApp.MainThreadScheduler)
  .Subscribe(x => CanDoIt = x);
```

#### Why?

 - `ObservableAsPropertyHelper` will take care of raising `INotifyPropertyChanged`
   events - if you're creating read-only properties, this can save so much boilerplate
   code.
 - `ObservableAsPropertyHelper` will use `MainThreadScheduler` to schedule subscribers,
  unless specified otherwise - no need to remember to do this yourself.
 - `WhenAny` lets you combine multiple properties, treat their changes as observable
  streams, and craft ViewModel-specific outputs.