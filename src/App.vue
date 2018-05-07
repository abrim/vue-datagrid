<template>
  <div id="app">
    <DataGrid :width="600" :height="500" :columns="columns" :prefs="prefs"/>
  </div>
</template>

<script>
import DataGrid from './components/DataGrid'
import DGTextCell from './components/DGTextCell'

export default {
  name: 'App',
  data () {
    return {
      prefs: {
        rowHeight: 30,

        readPageSize: 32, // number of rows to read at a time
        readRows: function (start, size) {
          return new Promise(function (resolve) {
            let rows = []
            for (let i = 0; i < size; ++i) {
              let r = {}
              if (start + i < 1000) {
                for (let j = 0; j < 20; ++j) {
                  r['col' + j] = '(' + (i + start) + ', ' + j + ')'
                }
                rows.push(r)
              } else {
                break
              }
            }
            resolve(rows)
          })
        }
      }
    }
  },
  computed: {
    columns () {
      let r = []
      for (let i = 0; i < 20; ++i) {
        r.push({
          title: 'Column ' + i,
          width: 80,
          key: 'col' + i,
          // getValue: row => (row || {})['col' + i],
          // setValue: function (row, v) {
          //   let r = (row || {})
          //   let ov = r['col' + i]
          //   if (ov !== v) {
          //     r.$dg = (r.$dg || {})
          //     r.$dg.old = (r.$dg.old || {})
          //     if (r.$dg.old['col' + i] === undefined) {
          //       r.$dg.old['col' + i] = ov
          //       r.$dg.dirty = true
          //     }
          //   }
          //   r['col' + i] = v
          //   return row
          // },
          component: DGTextCell
        })
      }
      return r
    }
  },
  components: {
    DataGrid
  }
}
</script>

<style>
#app {

  height: 100%;
}
</style>
