# Binding

Once you have created a command and exposed it from your view model, the next logical step is to consume it from your view. The most common means of achieving this is via the `BindCommand` method. This method - of which there are several overloads - is responsible for tying any source `ICommand` to a target control. Typical usage looks like this:

```cs
// in a view
this.BindCommand(
    this.ViewModel,
    x => x.MyCommand,
    x => x.myControl);
```