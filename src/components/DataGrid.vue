<template>
  <div class="dg-container" :style="styleSize(this.width, this.height)" ref="dgContainer">
    <div class="dg-topcontainer"
    :style="topContainerPosition">
      <div class="dg-columnheadercontainer" ref="columnHeaderContainer"
      :style="columnHeaderContainerPosition">
        <div class="dg-columnheader"
        v-for="(c,i) in displayColumns" :key="i"
        v-if="columnLefts[i + 1] > scrollX && columnLefts[i] - scrollX < scrollPaneWidth"
        :style="stylePosition(columnLefts[i] - scrollX, 0, displayColumns[i].width, prefs.columnHeaderHeight)">
          <component :is="rowHeaderComponent"
          :height="prefs.columnHeaderHeight"
          :width="displayColumns[i].width"
          :value="c.title" />
        </div>
      </div>
    </div>
    <div class="dg-leftcontainer"
    :style="leftContainerPosition">
      <div class="dg-rowheadercontainer"
      :style="rowHeaderContainerPosition">
        <div class="dg-rowheader"
        v-for="r in visibleRows" :key="r"
        v-if="startDisplayRow + r <= actTableLength"
        :style="stylePosition(0, (r - 1) * rowHeight - rowOffset, leftWidth, rowHeight)">
          <component :is="rowHeaderComponent"
          :height="rowHeight"
          :width="leftWidth"
          :value="r + startDisplayRow" />
        </div>
      </div>
    </div>
    <div class="dg-scrollpane" ref="scrollPane"
    :style="scrollpanePosition">
      <div class="dg-row"
      v-for="r in visibleRows" :key="r + startDisplayRow"
      :style="stylePosition(-scrollX, (r - 1) * rowHeight - rowOffset, columnLefts[displayColumns.length], rowHeight)"
      v-if="startDisplayRow + r <= actTableLength">
        <div class="dg-cell"
        v-for="(c, i) in displayColumns" :key="(r + startDisplayRow) + '-' + i"
        v-if="columnLefts[i + 1] > scrollX && columnLefts[i] - scrollX < scrollPaneWidth"
        :style="stylePosition(columnLefts[i], 0, displayColumns[i].width, rowHeight)"
         >
          <component :is="displayColumns[i].component"
          v-if="editRow !== startDisplayRow + r - 1 || editCol !== i"
          :width="displayColumns[i].width"
          :height="rowHeight"
          :value="displayColumns[i].value(tableData[startDisplayRow + r - 1])"
          @click.native="cellClick($event, startDisplayRow + r - 1, i)" />

          <component :is="displayColumns[i].editComponent"
          v-else
          :width="displayColumns[i].width"
          :height="rowHeight"
          :value="displayColumns[i].value(tableData[startDisplayRow + r - 1])"
          @blur.native="editBlur($event, startDisplayRow + r - 1, i)"
          @keydown="editKeyDown($event, startDisplayRow + r - 1, i)"/>
        </div>
      </div>
    </div>
    <div class="dg-rightcontainer"
    :style="rightContainerPosition">
      <div class="dg-verticalscroll" ref="verticalScroll"
      :style="{width: scrollbarWidth + 'px'}"
      @scroll="scrollpaneScrollY($event)">
        <div class="dg-endmarker"
        :style="stylePosition(0, estTableLength * rowHeight - 1, 1, 1)">
        </div>
      </div>
    </div>
    <div class="dg-bottomcontainer"
    :style="bottomContainerPosition">
      <div class="dg-footercontainer"
      :style="footerContainerPosition">
        <div class="dg-horizontalscroll" ref="horizontalScroll"
        :style="{height: scrollbarHeight + 'px'}"
        @scroll="scrollpaneScrollX($event)">
          <div class="dg-endmarker"
          :style="stylePosition(columnLefts[displayColumns.length] - 1, 0, 1, 1)">
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import DGTextCell from './DGTextCell'
import DGTextEditor from './DGTextEditor'

function styleSize (wid, hgt) {
  return {
    width: wid + 'px',
    height: hgt + 'px'
  }
}

function stylePosition (left, top, wid, hgt) {
  return {
    left: left + 'px',
    top: top + 'px',
    ...styleSize(wid, hgt)
  }
}

export default {
  data () {
    return {
      scrollbarWidth: 20,
      scrollbarHeight: 20,
      displayColumns: [],
      topHeight: 0, // top section height
      leftWidth: 80, // left section width
      rightWidth: 20, // right section width
      bottomHeight: 20, // bottom section height
      rowHeight: 40, // row height
      scrollX: 0,
      scrollY: 0,
      tableData: [],
      estTableLength: this.prefs.readPageSize * 2,
      actTableLength: Number.MAX_SAFE_INTEGER,
      rowHeaderComponent: undefined,
      columnHeaderComponent: undefined,
      columnHeaderHeight: 0,

      editRow: undefined,
      editCol: undefined
    }
  },
  props: {
    width: {
      type: Number,
      default: 600
    },
    height: {
      type: Number,
      default: 400
    },
    columns: {
      type: Array
    },
    prefs: {
      type: Object
    }
  },
  computed: {
    scrollpanePosition () {
      return stylePosition(this.leftWidth, this.topHeight, this.scrollPaneWidth, this.scrollPaneHeight)
    },
    topContainerPosition () {
      return stylePosition(0, 0, this.width, this.topHeight)
    },
    leftContainerPosition () {
      return stylePosition(0, this.topHeight, this.leftWidth, this.scrollPaneHeight)
    },
    rightContainerPosition () {
      return stylePosition(this.width - this.rightWidth, this.topHeight, this.rightWidth, this.scrollPaneHeight)
    },
    bottomContainerPosition () {
      return stylePosition(0, this.height - this.bottomHeight, this.width, this.bottomHeight)
    },
    columnHeaderContainerPosition () {
      return stylePosition(this.leftWidth, 0, this.scrollPaneWidth, this.topHeight)
    },
    rowHeaderContainerPosition () {
      return stylePosition(0, 0, this.leftWidth, this.scrollPaneHeight)
    },
    footerContainerPosition () {
      return stylePosition(this.leftWidth, 0, this.scrollPaneWidth, this.bottomHeight)
    },
    columnLefts () {
      let r = []
      let l = 0
      for (let i = 0; i < this.displayColumns.length; ++i) {
        r[i] = l
        l = l + this.displayColumns[i].width
      }
      r[this.displayColumns.length] = l
      return r
    },
    visibleRows () {
      return Math.floor((this.height - this.topHeight - this.bottomHeight) / this.rowHeight) + 1
    },
    startDisplayRow () {
      return Math.floor(this.scrollY / this.rowHeight)
    },
    rowOffset () {
      return this.scrollY % this.rowHeight
    },
    scrollPaneWidth () {
      return this.width - this.leftWidth - this.rightWidth
    },
    scrollPaneHeight () {
      return this.height - this.topHeight - this.bottomHeight
    }
  },
  methods: {
    styleSize: styleSize,
    stylePosition: stylePosition,
    containerWidth () {
      console.log('width')
      let r = this.$refs.dgContainer
      if (r) {
        console.log(r.clientWidth)
        return r.clientWidth
      }
      return 0
    },
    containerHeight () {
      let r = this.$refs.dgContainer
      if (r) {
        return r.clientHeight
      }
      return 0
    },
    readPage (toprow) {
      // console.log('* ', toprow)
      let tr = toprow - toprow % this.prefs.readPageSize
      for (let i = tr; i < toprow + this.visibleRows; i += this.prefs.readPageSize) {
        if (this.tableData[i] === undefined) {
          console.log(i)
          this.tableData[i] = {$loading: true}
          let self = this
          let p = this.prefs.readRows(i, this.prefs.readPageSize)
          p.then(function (res) {
            if (res.length < self.prefs.readPageSize) {
              if (res.length === 0) {
                self.estTableLength = Math.min(i, self.estTableLength, self.actTableLength)
                self.tableData.splice(self.estTableLength)
              } else {
                self.estTableLength = self.actTableLength = i + res.length
              }
            } else {
              self.estTableLength = Math.min(Math.max(i + res.length * 2, self.estTableLength), self.actTableLength)
            }
            self.tableData.splice(i, res.length, ...res)
          })
        }
      }
    },
    setScrollY (y) {
      this.scrollY = y
      this.$refs.verticalScroll.scrollTop = y
    },
    setScrollX (x) {
      this.scrollX = x
      this.$refs.horizontalScroll.scrollLeft = x
    },
    editCell (row, col) {
      if (row >= 0 && row < this.estTableLength && col >= 0 && col < this.displayColumns.length) {
        this.editRow = row
        this.editCol = col
        if (row <= this.startDisplayRow) {
          this.setScrollY(row * this.rowHeight)
        } else if ((row + 1) * this.rowHeight > this.scrollY + this.scrollPaneHeight) {
          this.setScrollY((row + 1) * this.rowHeight - this.scrollPaneHeight)
        }
        if (this.columnLefts[col] < this.scrollX) {
          this.setScrollX(this.columnLefts[col])
        } else if (this.columnLefts[col + 1] > this.scrollX + this.scrollPaneWidth) {
          this.setScrollX(this.columnLefts[col + 1] - this.scrollPaneWidth)
        }
      } else {
        console.log('illegal cell edit: ', row, col)
      }
    },
    // Events
    scrollpaneScrollY (event) {
      this.scrollY = event.target.scrollTop
      this.readPage(Math.floor(this.scrollY / this.rowHeight))
    },
    scrollpaneScrollX (event) {
      this.scrollX = event.target.scrollLeft
    },
    cellClick (event, row, col) {
      console.log('click ', row, col)
      this.editCell(row, col)
    },
    editBlur (event, row, col) {
      console.log('blur', row, col)
      if (this.editRow === row && this.editCol === col) {
        this.editRow = this.editCol = undefined
      }
    },
    editKeyDown (event, row, col) {
      switch (event.key) {
        case 'ArrowUp':
          if (row > 0) {
            this.editCell(row - 1, col)
          }
          break
        case 'ArrowDown':
          if (row + 1 < this.estTableLength) {
            this.editCell(row + 1, col)
          }
          break
        case 'Enter':
          if (col + 1 < this.displayColumns.length) {
            this.editCell(row, col + 1)
          } else if (row + 1 < this.estTableLength) {
            this.editCell(row + 1, 0)
          }
          break
        case 'Tab':
          if (event.shiftKey) {
            if (col > 0) {
              this.editCell(row, col - 1)
            } else if (row > 0) {
              this.editCell(row - 1, this.displayColumns.length - 1)
            }
          } else {
            if (col + 1 < this.displayColumns.length) {
              this.editCell(row, col + 1)
            } else if (row + 1 < this.estTableLength) {
              this.editCell(row + 1, 0)
            }
          }
          break
      }
    }
  },
  // Livecycle EVENTS
  created () {
    this.columnHeaderHeight = this.prefs.columnHeaderHeight
    this.topHeight = this.columnHeaderHeight
    this.rowHeaderComponent = this.prefs.rowHeaderComponent || DGTextCell
    this.columnHeaderComponent = this.prefs.columnHeaderComponent || DGTextCell
    let dc = []
    for (let i = 0; i < this.columns.length; ++i) {
      if (!this.columns[i].hidden) {
        let c = {
          component: DGTextCell,
          editComponent: DGTextEditor,
          ...this.columns[i]
        }
        dc.push(c)
      }
    }
    this.displayColumns = dc
  },
  updated () {
    this.scrollbarHeight = this.$refs.horizontalScroll.offsetHeight - this.$refs.horizontalScroll.clientHeight
    this.scrollbarWidth = this.$refs.verticalScroll.offsetWidth - this.$refs.verticalScroll.clientWidth
    this.bottomHeight = this.scrollbarHeight
    this.rightWidth = this.scrollbarWidth
  },
  mounted () {
    this.readPage(this.startDisplayRow)
    this.scrollbarHeight = this.$refs.horizontalScroll.offsetHeight - this.$refs.horizontalScroll.clientHeight
    this.scrollbarWidth = this.$refs.verticalScroll.offsetWidth - this.$refs.verticalScroll.clientWidth
    this.bottomHeight = this.scrollbarHeight
    this.rightWidth = this.scrollbarWidth
  }
}
</script>

<style>
div {
  /* outline: 1px dotted blue; */
}

.dg-container {
  position: relative;
}

.dg-scrollpane {
  position: absolute;
  overflow: hidden;
}

.dg-topcontainer {
  position: absolute;
  overflow: hidden;
}

.dg-leftcontainer {
  position: absolute;
  overflow: hidden;
}

.dg-rightcontainer {
  position: absolute;
  overflow: hidden;
}

.dg-bottomcontainer {
  position: absolute;
  overflow: hidden;
}

.dg-columnheadercontainer {
  position: absolute;
  overflow: hidden;
  white-space: nowrap;
}

.dg-columnheader {
  position: absolute;
  background-color: lightgray;
  overflow: hidden;
  border-right: 1px solid darkgray;
  border-bottom: 1px solid darkgray;
  text-align: center;
}

.dg-rowheadercontainer {
  position: absolute;
}

.dg-rowheader {
  position: absolute;
  background-color: lightgray;
  border-right: 1px solid darkgray;
  border-bottom: 1px solid darkgray;
  text-align: center;
}

.dg-endmarker {
  position: absolute;
}

.dg-row {
  position: absolute;
}

.dg-cell {
  position: absolute;
  border-right: 1px solid darkgray;
  border-bottom: 1px solid darkgray;
}

.dg-footercontainer {
  position: absolute;
}

.dg-horizontalscroll {
  overflow-x: auto;
  width: 100%;
  position: absolute;
  min-height: 4px;
  max-height: 20px;
  overflow-y: hidden;
}

.dg-verticalscroll {
  overflow-y: auto;
  height: 100%;
  position: absolute;
  max-width: 20px;
  overflow-x: hidden;
}
</style>
