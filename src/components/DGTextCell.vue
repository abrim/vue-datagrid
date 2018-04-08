<template>
  <div class="dg-textcell"
  :style="{width: width+'px', height: height+'px', lineHeight: (height-8)+'px'}">
  {{ value }}
  </div>
</template>

<script>
export default {
  props: ['width', 'height', 'value']
}
</script>

<style>
.dg-textcell {
  text-overflow: ellipsis;
  vertical-align: middle;
  padding: 4px;
  overflow: hidden;
}
</style>
