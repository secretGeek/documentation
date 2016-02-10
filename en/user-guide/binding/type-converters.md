# Type Converters

Inline type conversions for bindings were recently added in https://github.com/reactiveui/ReactiveUI/pull/1052


# Links
* https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI/PropertyBinding.cs#L50
* https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI/Xaml/BindingTypeConverters.cs#L24

* http://stackoverflow.com/questions/23592231/how-do-i-register-an-ibindingtypeconverter-in-reactiveui
* https://github.com/reactiveui/ReactiveUI/commit/7fba662c7308db60cfda9e6fb3331b7cb514f14c


## How GetAffinityForObjects works

Return 0 if you can't convert an object and return anything else if it is possible. The largest value (usually 100) ) wins. 

    public int GetAffinityForObjects(Type fromType, Type toType)
    {
        if (fromType == typeof(string))
        {
            return 100; // any number other than 0 signifies conversion is possible.
        }
        return 0;
    }

## How TryConvert works

    public bool TryConvert(object from, Type toType, object conversionHint, out object result)
    {
        try
        {
            result = !String.IsNullOrWhiteSpace(from);
        }
        catch (Exception ex)
        {
            this.Log().WarnException("Couldn't convert object to type: " + toType, ex);
            result = null;
            return false;
        }
    
        return true;
    }

## Registration

    Locator.CurrentMutable.RegisterConstant(
        new MyCoolTypeConverter(), typeof(IBindingTypeConverter));

## Usage

In most cases, usage is automatic.  When a binding is created, the converter(s) with the highest affinity are chosen from those registered with Splat. Optionally, you can provide a specific converter to override what the default would have been for either VM to View or View to VM. 

    this.Bind(ViewModel, vm => vm.ViewModelProperty, v => v.Control.Property, new CustomTypeConverter());
    
