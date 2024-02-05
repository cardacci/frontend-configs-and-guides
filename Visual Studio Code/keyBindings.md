To create key bindings: Go to _File_ > _Preferences_ > _Keyboard Shortcuts_

Then click on the following button (Open keyboard shortcuts).

![keyBindings-1.png](/assets/images/visual-studio-code/keyBindings-1.png)

The **keybindings.json** file will open. Within it, you can write the associations.


```
[
	{
		"args": {
			"snippet": "console.log('${TM_SELECTED_TEXT}', ${TM_SELECTED_TEXT});"
		},
		"command": "editor.action.insertSnippet",
		"key": "ctrl+shift+l",
		"when": "editorTextFocus"
	}
]
```

In this case, when we press Ctrl+Shift+L, we will achieve the desired result. It will write the console.log with the selected text and its description.
