# Model view viewmodel (MVVM)

Model-View-ViewModel, or MVVM for short, is a design pattern that is a variation of Martin Fowler's Presentation Model pattern which abstracts a view's state and behavior making view independent of any specific UI framework. The pattern promotes event-driven programming of user interfaces, and declarative property and command binding.

![](https://i.stack.imgur.com/yDjEr.png)



* Model - provides a domain model to some state content or to a data-layer representing content in a data-centric approach.

* View - Native, platform specific user interface implementation. ie. Xamarin.Android, Xamarin.iOS, Xamarin.Mac, WPF, Universal Windows Platform.

* View Model - a "model of the view", an abstraction of View which uses platform specific binder to mediate communication between View and Model.

http://wp.qmatteoq.com/the-mvvm-pattern-introduction/
