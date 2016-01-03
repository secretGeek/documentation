# Version 7.0

## Platform-independent

* `ReactiveCommand` re-written to improve behavior and APIs [#954](https://github.com/reactiveui/ReactiveUI/issues/954) [#955](https://github.com/reactiveui/ReactiveUI/issues/955) [#727](https://github.com/reactiveui/ReactiveUI/issues/727)
* `UserError` deprecated and replaced with the more flexible `UserInteraction` [#920](https://github.com/reactiveui/ReactiveUI/issues/920)
* removed fallback value support from bindings [#848](https://github.com/reactiveui/ReactiveUI/issues/848)
* removed binding overloads with implicit view resolution [#846](https://github.com/reactiveui/ReactiveUI/issues/846)
* fixed `WhenAnyObservable` bug that could result in a `NullReferenceException` [#830](https://github.com/reactiveui/ReactiveUI/issues/830)
* `ToProperty` now provides the initial value immediately [#815](https://github.com/reactiveui/ReactiveUI/issues/815)
* `CreateCollection` now takes an optional scheduler [#1009](https://github.com/reactiveui/ReactiveUI/issues/1009)

## Xamarin Forms

* `ViewModelViewHost` now extends `ContentView`, not `StackLayout` [#947](https://github.com/reactiveui/ReactiveUI/issues/947)
* fixed `RoutedViewHost` bugs [#890](https://github.com/reactiveui/ReactiveUI/issues/890)
* fixed `ViewModelViewHost` bugs [#946](https://github.com/reactiveui/ReactiveUI/issues/946) [#891](https://github.com/reactiveui/ReactiveUI/issues/891)
* added `ReactiveTextCell`, `ReactiveEntryCell`, `ReactiveSwitchCell`, `ReactiveImageCell`, and `ReactiveViewCell` [#882](https://github.com/reactiveui/ReactiveUI/issues/882)
* fixed events NuGet package to support Xamarin Forms [#881](https://github.com/reactiveui/ReactiveUI/issues/881)
* enhanced activation support [#949](https://github.com/reactiveui/ReactiveUI/issues/949)

## iOS

* added `ReactiveSplitViewController` [#899](https://github.com/reactiveui/ReactiveUI/issues/899)
* `ViewModelViewHost` rewritten [#868](https://github.com/reactiveui/ReactiveUI/issues/868)
* rewrote `ReactiveTableViewSource` and supporting infrastructure to fix several deficiencies [#740](https://github.com/reactiveui/ReactiveUI/issues/740) [#808](https://github.com/reactiveui/ReactiveUI/issues/808) [#867](https://github.com/reactiveui/ReactiveUI/issues/867)

## XAML

* `ViewModelViewHost` now includes a scalar `ViewContract` property [#870](https://github.com/reactiveui/ReactiveUI/issues/870)
* include `XmlnsDefinition` for supported platforms [#863](https://github.com/reactiveui/ReactiveUI/issues/863)