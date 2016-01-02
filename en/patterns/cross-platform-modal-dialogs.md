# Cross-platform Modal Dialogs


```csharp
public class AlertDialog : IYesNoDialog
{
    public IObservable<bool> Prompt(string title, string description)
    {
        var dlgDelegate = new UIAlertViewDelegateRx();
        var dlg = new UIAlertView(title, description, dlgDelegate, "No", "Yes");
        dlg.Show();

        return dlgDelegate.ClickedObs
            .Take(1)
            .Select(x => x.Item2 == 1);
    }
}
```
