<template>
  <div class="dg-container" :style="styleSize(this.width, this.height)" ref="dgContainer"
  @wheel="mouseWheel">
    <div class="dg-vertical_container"
    :style="{height: topOuterHeight + 'px'}"
    v-if="topOuterHeight > 0">
    </div>
    <div class="dg-vertical_container"
    :style="{height: innerHeight + 'px'}">
      <div class="dg-horizontal_container"
      :style="{width: leftOuterWidth + 'px'}"
      v-if="leftOuterWidth > 0">
    </div><div class="dg-horizontal_container"
      :style="{width: innerWidth + 'px'}">
        <div class="dg-vertical_container"
        :style="{height: topInnerHeight + 'px'}">
          <div class="dg-horizontal_container"
          :style="{width: leftInnerWidth + 'px'}">
          </div><div class="dg-horizontal_container"
          :style="{width: scrollPaneWidth + 'px'}">
            <div class="dg-columnheadercontainer" ref="columnHeaderContainer"
            :style="columnHeaderContainerPosition">
              <div class="dg-columnheader"
              v-for="(c,i) in displayColumns" :key="i"
              v-if="columnLefts[i + 1] > scrollX && columnLefts[i] - scrollX < scrollPaneWidth"
              :style="stylePosition(columnLefts[i] - scrollX, 0, displayColumns[i].width, actPrefs.header.column.height)">
                <component :is="actPrefs.header.column.component"
                :height="actPrefs.header.column.height"
                :width="displayColumns[i].width"
                :value="c.title" />
              </div>
            </div>
          </div><div class="dg-horizontal_container"
          :style="{width: rightInnerWidth + 'px'}">
          </div>
        </div>
        <div class="dg-vertical_container"
        :style="{height: scrollPaneHeight + 'px'}">
          <div class="dg-horizontal_container"
          :style="{width: leftInnerWidth + 'px'}">
            <div class="dg-rowheadercontainer"
            :style="rowHeaderContainerPosition">
              <div class="dg-rowheader"
              v-for="r in visibleRows" :key="r"
              v-if="startDisplayRow + r <= actTableLength"
              :style="stylePosition(0, (r - 1) * actPrefs.rowHeight - rowOffset, leftInnerWidth, actPrefs.rowHeight)">
                <component :is="actPrefs.header.row.component"
                :height="actPrefs.rowHeight"
                :width="leftInnerWidth"
                :value="r + startDisplayRow" />
              </div>
            </div>
          </div><div class="dg-horizontal_container" ref="scrollPane"
          :style="{width: scrollPaneWidth + 'px'}">
            <div class="dg-row"
            v-for="r in visibleRows" :key="r + startDisplayRow"
            :style="stylePosition(-scrollX, (r - 1) * actPrefs.rowHeight - rowOffset, columnLefts[displayColumns.length], actPrefs.rowHeight)"
            v-if="startDisplayRow + r <= actTableLength">
              <div class="dg-cell"
              v-for="(c, i) in displayColumns" :key="(r + startDisplayRow) + '-' + i"
              v-if="columnLefts[i + 1] > scrollX && columnLefts[i] - scrollX < scrollPaneWidth"
              :style="stylePosition(columnLefts[i], 0, displayColumns[i].width, actPrefs.rowHeight)" >
                <component :is="displayColumns[i].component"
                v-if="editRow !== startDisplayRow + r - 1 || editCol !== i"
                :width="displayColumns[i].width"
                :height="actPrefs.rowHeight"
                :value="displayColumns[i].getValue(tableData[startDisplayRow + r - 1])"
                @click.native="cellClick($event, startDisplayRow + r - 1, i)" />

                <component :is="displayColumns[i].editComponent"
                v-else
                :width="displayColumns[i].width"
                :height="actPrefs.rowHeight"
                @blur.native="editBlur($event, startDisplayRow + r - 1, i)"
                @keydown="editKeyDown($event, startDisplayRow + r - 1, i)"/>
              </div>
            </div>
          </div><div class="dg-horizontal_container"
          :style="{width: rightInnerWidth + 'px'}"
          v-if="rightInnerWidth > 0">
          </div>
        </div>
        <div class="dg-vertical_container"
        :style="{height: bottomInnerHeight + 'px'}"
        v-if="bottomInnerHeight > 0">
        </div>
        <div class="dg-horizontalscroll" ref="horizontalScroll"
        :style="{height: scrollbarHeight + 'px'}"
        @scroll="scrollpaneScrollX($event)">
          <div class="dg-endmarker"
          :style="stylePosition(columnLefts[displayColumns.length] - 1 + innerWidth - scrollPaneWidth, 0, 1, 1)">
          </div>
        </div>
      </div><div class="dg-verticalscroll" ref="verticalScroll"
      :style="{width: scrollbarWidth + 'px', height: (innerHeight - scrollbarHeight) + 'px'}"
      @scroll="scrollpaneScrollY($event)"
      v-if="scrollbarWidth > 0">
        <div class="dg-endmarker"
        :style="stylePosition(0, estTableLength * actPrefs.rowHeight - 1 + innerHeight - scrollbarHeight - scrollPaneHeight, 1, 1)">
        </div>
      </div><div class="dg-horizontal_container"
      :style="{width: rightOuterWidth + 'px'}"
      v-if="rightOuterWidth > 0">
      </div>
    </div>
    <div class="dg-vertical_container"
    :style="{height: bottomOuterHeight + 'px'}"
    v-if="bottomOuterHeight > 0">
    </div>

  </div>
</template>

<script>
import DGTextCell from './DGTextCell'
import DGTextEditor from './DGTextEditor'

/**
 * Simple object check.
 * @param item
 * @returns {boolean}
 */
export function isObject (item) {
  return (item && typeof item === 'object' && !Array.isArray(item))
}

/**
 * Deep merge two objects.
 * @param target
 * @param ...sources
 */
function mergeDeep (target, ...sources) {
  if (!sources.length) return target
  const source = sources.shift()

  if (isObject(target) && isObject(source)) {
    for (const key in source) {
      if (isObject(source[key])) {
        if (!target[key]) Object.assign(target, { [key]: {} })
        mergeDeep(target[key], source[key])
      } else {
        Object.assign(target, { [key]: source[key] })
      }
    }
  }
  return mergeDeep(target, ...sources)
}

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
      actPrefs: {
        rowHeight: 30,
        grid: {
          horizontal: {
            width: 1,
            style: 'solid',
            color: 'darkgray'
          },
          vertical: {
            width: 1,
            style: 'solid',
            color: 'darkgray'
          }
        },
        header: {
          row: {
            width: 60,
            component: undefined
          },
          column: {
            height: 30,
            component: undefined
          }
        },
        data: {
          input: {
            pageSize: 32,
            readPage: undefined
          }
        }
      },

      scrollbarWidth: 20,
      scrollbarHeight: 20,
      displayColumns: [],
      topInnerHeight: 0,
      topOuterHeight: 0,
      leftInnerWidth: 60,
      leftOuterWidth: 0,
      rightInnerWidth: 0,
      rightOuterWidth: 0,
      bottomInnerHeight: 0,
      bottomOuterHeight: 0,
      scrollX: 0,
      scrollY: 0,
      tableData: [],
      estTableLength: this.prefs.readPageSize * 2,
      actTableLength: Number.MAX_SAFE_INTEGER,

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
    innerHeight () {
      return this.height - this.topOuterHeight - this.bottomOuterHeight
    },
    innerWidth () {
      return this.width - this.leftOuterWidth - this.rightOuterWidth - this.scrollbarWidth
    },
    columnHeaderContainerPosition () {
      return stylePosition(0, 0, this.scrollPaneWidth, this.topInnerHeight)
    },
    rowHeaderContainerPosition () {
      return stylePosition(0, 0, this.leftInnerWidth, this.scrollPaneHeight)
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
      return Math.ceil(this.scrollPaneHeight / this.actPrefs.rowHeight) + 1
    },
    startDisplayRow () {
      return Math.floor(this.scrollY / this.actPrefs.rowHeight)
    },
    rowOffset () {
      return this.scrollY % this.actPrefs.rowHeight
    },
    scrollPaneWidth () {
      return this.width - this.leftInnerWidth - this.leftOuterWidth - this.rightInnerWidth - this.rightOuterWidth - this.scrollbarWidth
    },
    scrollPaneHeight () {
      return this.height - this.topInnerHeight - this.topOuterHeight - this.bottomInnerHeight - this.bottomOuterHeight - this.scrollbarHeight
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
          this.setScrollY(row * this.actPrefs.rowHeight)
        } else if ((row + 1) * this.actPrefs.rowHeight > this.scrollY + this.scrollPaneHeight) {
          this.setScrollY((row + 1) * this.actPrefs.rowHeight - this.scrollPaneHeight)
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
      this.readPage(Math.floor(this.scrollY / this.actPrefs.rowHeight))
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
      // BUG: commented out because it bugs out in chrome
      // if (this.editRow === row && this.editCol === col) {
      //   this.editRow = this.editCol = undefined
      // }
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
    },
    mouseWheel (event) {
      let dx = event.shiftKey ? event.deltaY : event.deltaX
      let dy = event.shiftKey ? event.deltaX : event.deltaY
      dx = Math.max(-this.scrollX, Math.min(this.columnLefts[this.displayColumns.length] - this.scrollPaneWidth - this.scrollX, dx))
      dy = Math.max(-this.scrollY, Math.min(this.estTableLength * this.actPrefs.rowHeight - this.scrollPaneHeight - this.scrollY, dy))
      this.setScrollX(this.scrollX + dx)
      this.setScrollY(this.scrollY + dy)
    }
  },
  // Livecycle EVENTS
  created () {
    let ap = {...this.prefs}
    this.actPrefs = mergeDeep(
      {
        rowHeight: 40,
        header: {
          column: {
            height: 40,
            component: DGTextCell
          },
          row: {
            width: 60,
            component: DGTextCell
          }
        }
      },
      ap
    )
    this.topInnerHeight = this.actPrefs.header.column.height
    this.leftInnerWidth = this.actPrefs.header.row.width
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
  },
  mounted () {
    this.readPage(this.startDisplayRow)
    this.scrollbarHeight = this.$refs.horizontalScroll.offsetHeight - this.$refs.horizontalScroll.clientHeight
    this.scrollbarWidth = this.$refs.verticalScroll.offsetWidth - this.$refs.verticalScroll.clientWidth
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

.dg-vertical_container {
  position: relative;
  overflow: hidden;
}

.dg-horizontal_container {
  position: relative;
  overflow: hidden;
  height: 100%;
  display: inline-block;
}

.dg-verticalscroll {
  overflow-y: auto;
  height: 100%;
  position: relative;
  overflow-x: hidden;
  display: inline-block;
  vertical-align: top;
}

.dg-horizontalscroll {
  overflow-x: auto;
  width: 100%;
  position: relative;
  overflow-y: hidden;
}

.dg-scrollpane {
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
</style>
