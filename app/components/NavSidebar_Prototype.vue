<template>
  <div>
    <UButton icon="i-lucide-menu" @click="collapsed = !collapsed" color="neutral" variant="ghost" class="m-2" />
    <UNavigationMenu
      :collapsed="collapsed"
      orientation="vertical"
      :items="items"
      class="h-full overflow-auto data-[orientation=vertical]:w-40 data-[orientation=vertical]:data-[collapsed=true]:w-16"
    />
      <!-- data-[orientation=vertical]:w-16 data-[orientation=vertical]:w-40  -->
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import type { NavigationMenuItem } from "@nuxt/ui";

const collapsed = ref(false);
const items = ref<NavigationMenuItem[][]>([
  [
    { label: "Home", icon: "i-lucide-home", to: "/" },
    { label: "About", icon: "i-lucide-info", to: "/about" },
  ],
  [
    { label: "Settings", icon: "i-lucide-settings", to: "/settings" },
    { label: "Help", icon: "i-lucide-help-circle", to: "/help" },
  ],
]);
</script>
