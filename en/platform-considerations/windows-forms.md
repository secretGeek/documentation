# Windows Forms

Example at https://github.com/shiftkey/reactiveui-winforms-example

Make sure you install `reactiveui-winforms` into your solution.

> I had one hell of a time getting started because I was missing the ReactiveUI-Winforms package...
> I actually wasn't missing the winforms-events package.  it was the "core" winforms package `ReactiveUI-Winforms`
> if you're missing that everything acts like it works, except it mostly doesn't.
> it was an extremely weird day figuring that out
> `OneWayBind` and `Bind` I couldn't tell if they worked at all.

Also see https://github.com/reactiveui/ReactiveUI/issues/997

Gluck (https://github.com/gluck) has published an unoffical forms package which from a fork of ReactiveUI repo:

https://github.com/gluck/ReactiveUI/commits/Net40-support

Note that reactiveui-*-events packages (neither the ones from Paul, nor Gluck's) have no adherence to ReactiveUI per-se, you can do RxUI without them, and you can use them without RxUI. They're simply "generated extension methods for every type exposing .Net events to expose corresponding IObservable wrappers", to save you the burden of writing the FromEvent/FromEventPattern yourself.

This winforms package contains helpers for every type in System.Windows.Forms.dll.