<template>
  <input type="text" autofocus class="dg-texteditor"
  :style="{width: width+'px', height: height+'px'}"
  ref="editor"
  @keydown="keyDown($event)"
  @focus="selectText" />
</template>

<script>
export default {
  props: ['width', 'height'],
  methods: {
    keyDown (event) {
      if (['ArrowUp', 'ArrowDown', 'Tab', 'Enter', 'PageUp', 'PageDown'].includes(event.key)) {
        this.$emit('keydown', event)
        event.preventDefault()
      }
    },
    selectText (event) {
      console.log('select')
      // event.target.select()
    }
  },
  updated () {
    console.log('updated')
  },
  mounted () {
    console.log('mounted')
    this.$refs.editor.focus()
    this.$refs.editor.select()
  }
}
</script>

<style>
.dg-texteditor {
  font: inherit;
  font-size: inherit;
  padding: 4px;
  border: 0px;
  outline: solid 1px blue;
  outline-offset: -1px;
}

</style>
