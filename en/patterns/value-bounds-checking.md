# Value Bounds Checking


Matt:
> Does anyone have a good way to deal with value bounds checking? for instance "username should be 3 to 12 characters" from the viewmodel.

Kent:
> my approach is normally to break into separate properties: `UserName` and `UserNameError` (which is `null` if there is no error). Then use the platform’s validation/error mechanism to bring to user’s attention

Brendan:
what Kent said
> excep we called it `UserNameValidator` and rolled a bit of interesting code to declaratively define the rules - and then had a `.Result` to inspect at the end

Dennis:
> I'm doing it exactly like @kentcb