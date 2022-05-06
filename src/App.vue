<template>
  <div id="app">
    <monaco-editor 
      isShowTextError
      :value="'c'"
      @input="onChange"
    />
  </div>
</template>

<script>
import monacoEditor from './components/VueCustomMonacoEditor.vue'
export default {
  name: 'App',
  components: { 'monaco-editor': monacoEditor  },
  methods: {
    onChange(value) {
      console.log(value)
    }
  }
}
</script>
