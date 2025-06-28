<script setup lang="ts">
import { ref, computed, watch, onBeforeUnmount, toRefs } from 'vue'

/**
 * A lightweight, drop-in replacement for Nuxt-UI-Pro `UDashboardSidebar`.
 *
 * Features implemented:
 *  • Collapsible (via prop or v-model:collapsed)
 *  • Resizable (drag handle – desktop only)
 *  • Slots – header, default (body) and footer
 *  • `side`, `minSize`, `maxSize`, `defaultSize`, `collapsedSize` props
 *
 * ⚠️  The public API purposely mirrors `UDashboardSidebar` so you can migrate
 *     to Nuxt-UI-Pro in the future with *zero* code changes.
 */

type Side = 'left' | 'right'

const props = withDefaults(defineProps<{
  /** Which side of the screen to pin the sidebar */
  side?: Side
  /** Allow the user to resize the sidebar by dragging the handle */
  resizable?: boolean
  /** Allow the user to fully collapse the sidebar */
  collapsible?: boolean
  /** Minimum width in rem (when resizable) */
  minSize?: number
  /** Maximum width in rem (when resizable) */
  maxSize?: number
  /** Default width in rem – initial value when not collapsed */
  defaultSize?: number
  /** Width in rem when collapsed – set to 0 to hide */
  collapsedSize?: number
  /** v-model:open – mobile only (slideover) */
  open?: boolean
  /** v-model:collapsed – desktop collapsed state */
  collapsed?: boolean
}>(), {
  side: 'left',
  resizable: false,
  collapsible: false,
  minSize: 10,
  maxSize: 20,
  defaultSize: 15,
  collapsedSize: 0,
  open: true,
  collapsed: false
})

const emit = defineEmits<{
  /** Sync open state */
  'update:open': [boolean]
  /** Sync collapsed state */
  'update:collapsed': [boolean]
  /** Fired while dragging – emits the size (rem) */
  resize: [number]
}>()

const size = ref(props.collapsed ? props.collapsedSize : props.defaultSize)
const isDragging = ref(false)
const collapsed = ref(props.collapsed)

watch(() => props.collapsed, v => {
  collapsed.value = v
  size.value = v ? props.collapsedSize : props.defaultSize
})

watch(collapsed, v => emit('update:collapsed', v))

function onPointerMove (e: PointerEvent) {
  if (!isDragging.value) return
  const movementX = props.side === 'left' ? e.clientX : window.innerWidth - e.clientX
  const pxPerRem = parseFloat(getComputedStyle(document.documentElement).fontSize)
  let next = movementX / pxPerRem // convert to rem
  next = Math.min(props.maxSize, Math.max(props.minSize, next))
  size.value = next
  emit('resize', next)

  if (props.collapsible) {
    if (next <= props.collapsedSize + 1) {
      collapsed.value = true
    } else {
      collapsed.value = false
    }
  }
}

function stopDragging () {
  if (!isDragging.value) return
  isDragging.value = false
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', stopDragging)
}

function startDragging (e: PointerEvent) {
  if (!props.resizable) return
  isDragging.value = true
  e.preventDefault()
  window.addEventListener('pointermove', onPointerMove)
  window.addEventListener('pointerup', stopDragging)
}

onBeforeUnmount(() => stopDragging())

const { side, resizable } = toRefs(props)

const sidebarStyle = computed(() => {
  const widthRem = collapsed.value ? props.collapsedSize : size.value
  return {
    width: `${widthRem}rem`,
    transition: isDragging.value ? 'none' : 'width 150ms ease'
  }
})
</script>

<template>
  <!-- Root wrapper -->
  <div
    class="flex flex-col shrink-0 bg-default border-default h-svh data-[collapsed=true]:items-center"
    :class="[
      side === 'left' ? 'border-r' : 'border-l',
      collapsed ? 'data-[collapsed=true]' : ''
    ]"
    :style="sidebarStyle"
    :data-collapsed="collapsed"
  >
    <!-- Header slot -->
    <div v-if="$slots.header" class="flex items-center gap-2 p-4">
      <slot name="header" :collapsed="collapsed" />
    </div>

    <!-- Default slot (content) -->
    <div class="flex flex-col flex-1 overflow-y-auto gap-4 p-4">
      <slot :collapsed="collapsed" />
    </div>

    <!-- Footer slot -->
    <div v-if="$slots.footer" class="p-4 mt-auto">
      <slot name="footer" :collapsed="collapsed" />
    </div>

    <!-- Resize handle -->
    <div
      v-if="resizable"
      class="w-1 cursor-ew-resize absolute top-0 bottom-0 z-10 opacity-0 hover:opacity-100 transition-opacity"
      :class="side === 'left' ? 'right-0' : 'left-0'"
      @pointerdown="startDragging"
    />
  </div>
</template>

<style scoped>
/***** Ensure min visible width when collapsedSize == 0 *****/
[data-collapsed="true"] {
  min-width: 4rem; /* same as min-w-16 Tailwind class */
}
</style>