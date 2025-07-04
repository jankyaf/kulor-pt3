<!--
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
-->

<template>
  <aside
    ref="sidebarRef"
    :class="[
      'relative flex flex-col h-screen flex-shrink-0 bg-white dark:bg-gray-900',
      'transition-[width] duration-200 ease-in-out',
      'border-gray-200 dark:border-gray-800',
      side === 'left' ? 'border-r' : 'border-l',
    ]"
    :style="sidebarStyle"
  >
    <!-- Content wrapper for handling overflow and transitions -->
    <div
      :class="[
        'flex flex-col h-full w-full overflow-hidden',
        'transition-opacity duration-200 ease-in-out',
        isCollapsed ? 'opacity-0 pointer-events-none' : 'opacity-100',
      ]"
    >
      <header v-if="$slots.header" class="flex-shrink-0 p-4">
        <slot name="header" :collapsed="isCollapsed" />
      </header>

      <div class="flex-grow overflow-y-auto p-4">
        <slot :collapsed="isCollapsed" />
      </div>

      <footer v-if="$slots.footer" class="flex-shrink-0 p-4">
        <slot name="footer" :collapsed="isCollapsed" />
      </footer>
    </div>

    <!-- Resize handle, only shown on desktop -->
    <div
      v-if="!isCollapsed"
      :class="[
        'absolute top-0 bottom-0 w-2 cursor-col-resize z-10',
        'hidden md:block', // Hide on mobile where resizing is impractical
        side === 'left' ? '-right-1' : '-left-1',
      ]"
      @mousedown.prevent="startResize"
    />
  </aside>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onBeforeUnmount } from 'vue';

//==============================================================================
// PROPS & EMITS
//==============================================================================

const props = defineProps({
  /** The side of the screen where the sidebar is displayed. */
  side: {
    type: String as () => 'left' | 'right',
    default: 'left',
    validator: (value: string) => ['left', 'right'].includes(value),
  },
  /** Controls the collapsed state of the sidebar. */
  collapsed: {
    type: Boolean,
    default: false,
  },
  /** The minimum width the sidebar can be resized to. */
  minSize: {
    type: Number,
    default: 200,
  },
  /** The maximum width the sidebar can be resized to. */
  maxSize: {
    type: Number,
    default: 400,
  },
  /** The initial width of the sidebar when not collapsed. */
  defaultSize: {
    type: Number,
    default: 250,
  },
  /** The width of the sidebar when it is collapsed. */
  collapsedSize: {
    type: Number,
    default: 64,
  },
});

const emit = defineEmits(['update:collapsed']);

//==============================================================================
// STATE
//==============================================================================

const sidebarRef = ref<HTMLElement | null>(null);
const isResizing = ref(false);
const isCollapsed = ref(props.collapsed);
const currentSize = ref(props.defaultSize);

//==============================================================================
// COMPUTED PROPERTIES
//==============================================================================

const sidebarStyle = computed(() => ({
  // Inline style is necessary for dynamic, pixel-perfect resizing
  width: `${
    isCollapsed.value ? props.collapsedSize : currentSize.value
  }px`,
}));

//==============================================================================
// RESIZE LOGIC
//==============================================================================

const startResize = (event: MouseEvent) => {
  isResizing.value = true;
  document.body.style.cursor = 'col-resize';
  document.body.style.userSelect = 'none';

  window.addEventListener('mousemove', doResize);
  window.addEventListener('mouseup', stopResize);
};

const doResize = (event: MouseEvent) => {
  if (!isResizing.value) return;

  let newWidth;
  if (props.side === 'left') {
    newWidth = event.clientX;
  } else {
    newWidth = window.innerWidth - event.clientX;
  }

  // Clamp the width between min and max size
  currentSize.value = Math.max(
    props.minSize,
    Math.min(newWidth, props.maxSize),
  );
};

const stopResize = () => {
  isResizing.value = false;
  document.body.style.cursor = '';
  document.body.style.userSelect = '';

  window.removeEventListener('mousemove', doResize);
  window.removeEventListener('mouseup', stopResize);
};

//==============================================================================
// WATCHERS & LIFECYCLE
//==============================================================================

watch(
  () => props.collapsed,
  (newValue) => {
    isCollapsed.value = newValue;
  },
);

watch(isCollapsed, (newValue) => {
  emit('update:collapsed', newValue);
});

onMounted(() => {
  currentSize.value = Math.max(
    props.minSize,
    Math.min(props.defaultSize, props.maxSize),
  );
});

onBeforeUnmount(() => {
  if (isResizing.value) {
    stopResize();
  }
});
</script>