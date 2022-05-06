<template>
  <div style="position: relative;" :class="{ 'monaco-editor-text-error': isEditorError && isShowErrors }" ref="editor" :style="style">
    <div v-if="isShowErrors" class="monaco-editor-error-area d-flex justify-center align-center">
      <span v-if="editorErrorText !== ''" style="color: red">{{ editorErrorText }}</span>
    </div>
  </div>
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
    isShowTextError: { type: Boolean, default: false },
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

  data: () => ({
    isEditorError: false,
    editorErrorText: ''
  }),

  mounted() {
    this.initMonaco()
  },

  beforeDestroy() {
    this.editor && this.editor.dispose()
  },

  computed: {
    isShowErrors() {
      if (this.isShowTextError && this.language === 'javascript') {
        return true
      }
      return false
    },

    style() {
      return {
        width: !/^\d+$/.test(this.width) ? this.width : `${this.width}px`,
        height: !/^\d+$/.test(this.height) ? this.height : `${this.height}px`
      }
    },

    defaultOptions() {
      return {
        language: this.language,
        value: this.value,
        autoIndent: true,
        formatOnPaste: true,
        formatOnType: true,
        ...this.options
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

    value() {
      this.editor && this.value !== this._getValue() && this._setValue(this.value)
    },

    theme() {
      this.editor && loader.__getMonacoInstance().editor.setTheme(this.theme)
    }
  },

  methods: {
    initMonaco() {
      const el = this.$refs.editor
      if (this.editor) {
        return
      }
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
      this._validateText(value)
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

    _validateText(value) {
      if (!this.isShowErrors) {
        return
      }
      this.isEditorError = false
      this.editorErrorText = ''
      try {
        new Function(value)
      } catch (error) {
        this.editorErrorText = error
        this.isEditorError = true
      }
    },

    _emitChange(value, event) {
      this._validateText(value)
      this.$emit('change', value, event)
      this.$emit('input', value)
    }
  }
}
</script>

<style scoped>
.monaco-editor-text-error >>> .margin-view-overlays  {
  border-bottom: 3px solid red;
  border-left: 3px solid red;
  border-top: 3px solid red;
}
.monaco-editor-text-error >>> .monaco-mouse-cursor-text {
  border-bottom: 3px solid red;
  border-top: 3px solid red;
}
.monaco-editor-text-error >>> .minimap-decorations-layer {
  border-top: 3px solid red;
  border-bottom: 3px solid red;
}
.monaco-editor-text-error >>> .decorationsOverviewRuler {
  border-top: 3px solid red;
  border-bottom: 3px solid red;
  border-right: 3px solid red;
}
.monaco-editor-error-area {
  position: absolute;
  z-index: 999;
  right: 0;
}
</style>
