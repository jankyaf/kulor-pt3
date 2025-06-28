<script setup lang="ts">
import { ref } from 'vue'
import NavSidebar from '~/components/NavSidebar.vue'

// Collapsed state shared with NavSidebar via v-model
const collapsed = ref(false)
</script>

<template>
  <div class="flex h-screen">
    <!-- Sidebar -->
    <NavSidebar
      v-model:collapsed="collapsed"
      resizable
      collapsible
      class="shadow-sm"
    >
      <!-- Toggle button in sidebar header -->
      <template #header="{ collapsed: isCollapsed }">
        <!-- Square ghost button toggling collapse -->
        <UButton
          :icon="isCollapsed ? 'i-lucide-chevron-right' : 'i-lucide-chevron-left'"
          color="neutral"
          variant="ghost"
          :square="true"
          @click="collapsed = !collapsed"
        />
      </template>
      <!-- Provide sidebar body (default slot) -->
      <UNavigationMenu
        orientation="vertical"
        :collapsed="collapsed"
        :items="[
          [{ label: 'Landing', icon: 'i-lucide-flag', to: '/' }],
          [{ label: 'Home', icon: 'i-lucide-home', to: '/home' }]
        ]"
        class="flex-1"
      />
    </NavSidebar>

    <!-- Main content -->
    <div class="flex-1 overflow-auto">
      <slot />
    </div>
  </div>
</template>