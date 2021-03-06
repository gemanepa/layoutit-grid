<template>
  <span class="token string" v-for="(item, i) in trackSizesAndLineNames" :key="item.type + (item.pos || item.track)"
    >{{ separatorBeforeItem(i)
    }}<CssCodeTrackSize
      v-if="item.type === 'size'"
      :grid="grid"
      :type="type"
      :track="item.track"
      @move="onMove($event, i)"
      :el="item.el"
    /><CssCodeLineName
      v-if="item.type === 'line'"
      :grid="grid"
      :type="type"
      :pos="item.pos"
      @move="onMove($event, i)"
      :el="item.el"
    />{{ separatorAfterItem(i) }}
  </span>
</template>

<script setup="props, { emit }">
export { default as CssCodeTrackSize } from './CssCodeTrackSize.vue'
export { default as CssCodeLineName } from './CssCodeLineName.vue'
import { isValidTrackSize } from '../../store.js'
import { ref, computed } from 'vue'
import { debounce } from 'lodash-es'

import { namedTemplateColumns, namedTemplateRows, parseGridTemplate } from '../../utils.js'

export default {
  props: {
    grid: { type: Object, required: true },
    type: { type: String, required: true },
    repeat: { type: Boolean, default: false },
  },
}

export const multiline = computed(() => {
  const { lineNames } = props.grid[props.type]
  return lineNames.some((line) => line.name !== '' && line.active === true)
})

export function separatorBeforeItem(i) {
  return multiline.value && i === 0 ? '\n    ' : ''
}
export function separatorAfterItem(i) {
  const isLast = i === trackSizesAndLineNames.value.length - 1
  return trackSizesAndLineNames.value[i].type === 'size' && !isLast && multiline.value ? '\n    ' : isLast ? '' : ' '
}

export const trackSizesAndLineNames = computed(() => {
  const { sizes, lineNames } = props.grid[props.type]
  const items = []
  for (var i = 0; i < lineNames.length; i++) {
    const { active, name } = lineNames[i]
    if (active && name) {
      items.push({ type: 'line', pos: i, el: ref(null) })
    }
    if (i < sizes.length) {
      items.push({ type: 'size', track: i + 1, el: ref(null) })
    }
  }
  return items
})

export function onMove(event, i) {
  switch (event.action) {
    case 'right':
      if (i + 1 < trackSizesAndLineNames.value.length) {
        trackSizesAndLineNames.value[i + 1].el.value.focus()
      } else {
        if (document.activeElement) {
          document.activeElement.blur()
        }
      }
      break
    case 'left':
      if (i - 1 >= 0) {
        trackSizesAndLineNames.value[i - 1].el.value.focus()
      } else {
        if (document.activeElement) {
          document.activeElement.blur()
        }
      }
      break
  }
}
</script>

<style scoped lang="scss"></style>
