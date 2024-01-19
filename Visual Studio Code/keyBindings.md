Ir a _File > Preferences > Keyboard Shortcuts_

Luego hacer click en el siguiente botón (Open keyboard shortcuts)

![keyBindings-1.png](assets/images/keyBindings-1.png)

Se abrirá el archivo **keybindings.json**, dentro de él escribimos las asociaciones


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
