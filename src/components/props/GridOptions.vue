<template>
  <div class="grid-settings-container">
    <div class="items columns">
      <h2><span style="transform: rotate(90deg); display: inline-block">≑</span> Grid Columns</h2>
      <div class="inner-items">
        <div v-for="column in colsNumber" :key="column" :data-col="column">
          <input
            v-if="unitHasValue(getColUnit(grid, column - 1))"
            :value="getColValue(grid, column - 1)"
            :class="{
              active: isFocused('col', column),
              dragging: dragging && (dragging.colLine === column || dragging.colLine === column + 1),
            }"
            :type="getColUnit(grid, column - 1) === 'minmax' ? 'text' : 'number'"
            :aria-label="`column ${column} size`"
            min="0"
            step="0.5"
            @input="setColValue(grid, column - 1, $event.target.value)"
          />
          <UnitSelect
            :value="getColUnit(grid, column - 1)"
            type="grid"
            :aria-label="`column ${column} unit`"
            @input="onColUnitInput($event.target.value, column - 1)"
          />
          <OptionsButton
            class="remove-button"
            :aria-label="`remove column ${column}`"
            :disabled="grid.col.sizes.length === 1"
            @click="removeCol(grid, column - 1)"
          >
            <IconRemove />
          </OptionsButton>
        </div>
      </div>
      <OptionsButton class="add-button" @click="addCol(grid, '1fr')">Add</OptionsButton>
    </div>
    <div class="items rows">
      <h2><span>≑</span> Grid Rows</h2>
      <div class="inner-items">
        <div v-for="row in rowsNumber" :key="row" :data-row="row">
          <input
            v-if="unitHasValue(getRowUnit(grid, row - 1))"
            :value="getRowValue(grid, row - 1)"
            :class="{
              active: isFocused('row', row),
              dragging: dragging && (dragging.rowLine === row || dragging.rowLine === row + 1),
            }"
            :type="getRowUnit(grid, row - 1) === 'minmax' ? 'text' : 'number'"
            :aria-label="`row ${row} size`"
            min="0"
            step="0.5"
            @input="setRowValue(grid, row - 1, $event.target.value)"
          />
          <UnitSelect
            :value="getRowUnit(grid, row - 1)"
            type="grid"
            :aria-label="`row ${row} unit`"
            @input="onRowUnitInput($event.target.value, row - 1)"
          />
          <OptionsButton
            class="remove-button"
            :aria-label="`remove row ${row}`"
            :disabled="grid.row.sizes.length === 1"
            @click="removeRow(grid, row - 1)"
          >
            <IconRemove />
          </OptionsButton>
        </div>
      </div>
      <OptionsButton class="add-button" @click="addRow(grid, '1fr')">Add</OptionsButton>
    </div>
    <div class="items gaps">
      <h2><span>⊞</span> Grid Gap</h2>
      <GapInput :grid="grid" type="row" />
      <GapInput :grid="grid" type="col" />
    </div>
  </div>
</template>

<script setup="props">
export { default as IconRemove } from '../icons/IconRemove.vue'
export { default as UnitSelect } from '../common/UnitSelect.vue'
export { default as GapInput } from '../common/GapInput.vue'
export { default as OptionsButton } from '../basic/OptionsButton.vue'

export default {
  props: {
    grid: { type: Object, required: true },
  },
}

import { toRefs } from 'vue'

export {
  addCol,
  addRow,
  getRowValue,
  getRowUnit,
  setRowValue,
  removeRow,
  getColValue,
  getColUnit,
  setColValue,
  removeCol,
  dragging,
} from '../../store.js'

import { setRowValueUnit, setColValueUnit, trackFocus } from '../../store.js'
import { useGridDimensions } from '../../composables/area.js'

const { grid } = toRefs(props)

export const { colsNumber, rowsNumber } = useGridDimensions(grid)

export function unitHasValue(unit) {
  return !(unit === 'auto' || unit === 'min-content' || unit === 'max-content')
}

// TODO: compute new value using previous size

function defaultValueForUnit(unit) {
  switch (unit) {
    case 'px':
      return 300
    case 'fr':
      return 1
    case 'em':
      return 4
    case '%':
      return 10
    case 'minmax':
      return '20px, 60px'
    case 'auto':
    case 'min-content':
    case 'max-content':
      return ''
    default:
      return 1
  }
}

export function onRowUnitInput(unit, row) {
  setRowValueUnit(props.grid, row, { value: defaultValueForUnit(unit), unit })
}

export function onColUnitInput(unit, col) {
  setColValueUnit(props.grid, col, { value: defaultValueForUnit(unit), unit })
}

export function isFocused(type, track) {
  const tf = trackFocus.value
  return tf && tf.grid === grid.value && tf.type === type && tf.track === track
}
</script>

<style scoped lang="scss">
h2 {
  margin: 0 0 10px;
  font-family: 'Alegreya Sans', 'Helvetica Neue', Arial, sans-serif;
  margin-top: 10px;
  margin-bottom: 0.625em;
  font-size: 18.72px;
  span {
    font-weight: normal;
  }
}

.items {
  margin-bottom: 20px;
  .inner-items {
    max-height: 195px;
    overflow: auto;
    font-family: arial;
    > div {
      display: grid;
      grid-template-columns: 70px auto 30px;
      grid-gap: 0 5px;
      border-radius: 2px;
      height: 30px;
      margin-bottom: 6px;
      input {
        text-align: center;
        width: 100%;
        border: 0;
        border-radius: 2px;
        padding: 0.313em;
        font-size: 14px;
        background: #fff;
        color: #333;
        &.dragging {
          background: #bbe5b3;
        }
        &.active {
          background: #bbe5b3;
        }
      }

      select {
        background: #fff;
        border: 0;
        width: 100%;
        font-size: 14px;
        color: #333;
      }
    }
  }
}

.remove-button {
  width: 100%;
  padding: 8px 0;
  margin-bottom: 0;
  background: var(--color-remove);
  &:hover {
    background: var(--color-remove-active);
  }
  svg {
    padding: 1.2px;
    width: 100%;
    height: 100%;
    fill: #fff;
  }
}

.add-button {
  min-height: 2.188em;
  min-width: 2.188em;
  margin-bottom: 0.626em;
  padding: 0.313em 9px;
  background: var(--color-add);
  &:hover {
    background: var(--color-add-active);
  }
}
</style>
