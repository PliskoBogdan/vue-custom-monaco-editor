<template>
  <div ref="editor" :style="style"></div>
</template>

<script>
import loader from '@monaco-editor/loader'
function noop() {}

export default {
  props: {
    diffEditor: { type: Boolean, default: false },
    width: { type: [String, Number], default: '100%' },
    height: { type: [String, Number], default: '100%' },
    original: String,
    value: String,
    language: { type: String, default: 'javascript' },
    theme: { type: String, default: 'vs' },
    options: {
      type: Object,
      default() {
        return {}
      }
    },
    editorMounted: { type: Function, default: noop },
    editorBeforeMount: { type: Function, default: noop }
  },

  mounted() {
    this.initMonaco()
  },

  beforeDestroy() {
    this.editor && this.editor.dispose()
  },

  computed: {
    style() {
      return {
        width: !/^\d+$/.test(this.width) ? this.width : `${this.width}px`,
        height: !/^\d+$/.test(this.height) ? this.height : `${this.height}px`
      }
    }
  },

  watch: {
    options: {
      deep: true,
      handler(options) {
        this.editor && this.editor.updateOptions(options)
      }
    },

    style() {
      this.editor &&
        this.$nextTick(() => {
          this.editor.layout()
        })
    },

    language() {
      if (!this.editor) {
        return
      }
      const monaco = loader.__getMonacoInstance()
      if (this.diffEditor) {
        const { original, modified } = this.editor.getModel()
        monaco.editor.setModelLanguage(original, this.language)
        monaco.editor.setModelLanguage(modified, this.language)
      } else {
        monaco.editor.setModelLanguage(this.editor.getModel(), this.language)
      }
    },

    theme() {
      this.editor && loader.__getMonacoInstance().editor.setTheme(this.theme)
    }
  },

  methods: {
    initMonaco() {
      const el = this.$refs.editor
      loader.init().then((monaco) => {
        this.editor = monaco.editor[this.diffEditor ? 'createDiffEditor' : 'create'](el, {
          language: this.language,
          value: this.value,
          autoIndent: true,
          formatOnPaste: true,
          formatOnType: true,
          ...this.options
        })
        this._editorMounted(this.editor)
        this.diffEditor && this._setModel(this.value, this.original)
      })
    },
    _editorMounted(editor) {
      this.editorMounted(editor, loader.__getMonacoInstance())
      if (this.diffEditor) {
        editor.onDidUpdateDiff((event) => {
          const value = this._getValue()
          this._emitChange(value, event)
        })
      } else {
        editor.onDidChangeModelContent((event) => {
          const value = this._getValue()
          this._emitChange(value, event)
        })
      }
    },

    _setModel(value, original) {
      const { language } = this
      const originalModel = loader.__getMonacoInstance().editor.createModel(original, language)
      const modifiedModel = loader.__getMonacoInstance().editor.createModel(value, language)
      this.editor.setModel({
        original: originalModel,
        modified: modifiedModel
      })
    },

    _setValue(value) {
      let editor = this._getEditor()
      if (editor) {
        return editor.setValue(value)
      }
    },

    _getEditor() {
      if (!this.editor) {
        return null
      }
      return this.diffEditor ? this.editor.modifiedEditor : this.editor
    },

    _getValue() {
      let editor = this._getEditor()
      if (!editor) {
        return ''
      }
      return editor.getValue()
    },

    _emitChange(value, event) {
      this.$emit('change', value, event)
      this.$emit('input', value)
    }
  }
}
</script>