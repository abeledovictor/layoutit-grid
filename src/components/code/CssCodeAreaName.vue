<template>
  <span
    role="textbox"
    contenteditable
    spellcheck="false"
    :class="['input', { active: false }]"
    @keydown="onCodeInputKeydown"
    @input="onInput"
    >{{ areaName }}</span
  >
</template>

<script setup="props, { emit }">
import { dragging, currentArea, isValidAreaName } from '../../store.js'
import { computed, nextTick } from 'vue'
export { onCodeInputKeydown } from '../../utils.js'

export default {
  props: {
    area: { type: Object, required: true },
  },
}

export { currentArea }

export const areaName = computed({
  get: () => props.area.name,
  set: (str) => (props.area.name = str),
})

function textFrom(event) {
  const textNode = event.target.childNodes[0]
  return textNode && textNode.data
}

export function onInput(event) {
  areaNameChanged(event)
}

const areaNameChanged = (event) => {
  const text = textFrom(event)
  if (isValidAreaName(text)) {
    areaName.value = text
  }
}
</script>

<style scoped lang="scss">
span {
  &:hover {
    color: white;
  }
  &:focus {
    color: white;
  }
}
</style>
