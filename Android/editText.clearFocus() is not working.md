This is happening because your EditText is the first focusable view.

From the docs,

```
Note: When a View clears focus the framework is trying to give focus to 
the first focusable View from the top. Hence, if this View is the first 
from the top that can take focus, then all callbacks related to clearing focus 
will be invoked after which the framework will give focus to this view.
```
You can try setting a dummy focusable view above the EditText to clear the focus from it.
```
android:focusableInTouchMode="true"
```
