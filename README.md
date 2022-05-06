## Project
```
This component is similar to vue-monaco-editor, but with actual monaco-editor version.

```

## Props
If you specify `value` property, the component behaves in controlled mode.
Otherwise, it behaves in uncontrolled mode.

- `isShowTextError` - allows you to wrap the text in a function and look 
   for the presence of syntacticerrors. In case of errors, 
   highlights the monaco border and displays the error text `(ONLY JS CODE)`
- `width` width of editor. Defaults to `100%`.
- `height` height of editor. Defaults to `100%`.
- `value` value of the auto created model in the editor.
- `original` value of the auto created original model in the editor.
- `language` the initial language of the auto created model in the editor. Defaults to `javascript`.
- `theme` the theme of the editor. Defaults to `vs`.
- `options` refer to [Monaco interface IEditorConstructionOptions](https://microsoft.github.io/monaco-editor/api/modules/monaco.editor.html#EditorOptions).
- `editorBeforeMount(monaco)` The function called before the editor mounted (similar to `beforeMount` of Vue).
- `editorMounted(editor, monaco)` The function called when the editor has been mounted (similar to `mounted` of Vue).
- `change(newValue, event)` an event emitted when the content of the current model has changed.

### GitHub
See [Git Hub Project](https://github.com/PliskoBogdan/vue-custom-monaco-editor).
