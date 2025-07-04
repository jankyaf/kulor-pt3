<!-- pages/index.vue or components/MyDashboard.vue -->
<template>
  <!-- Set text color and background on a parent for the whole page -->
  <div
    class="bg-gray-50 dark:bg-gray-950 text-gray-900 dark:text-white font-sans"
  >
    <div class="flex flex-row">
      <DashboardSidebar
        v-model:collapsed="isSidebarCollapsed"
        :default-size="280"
        :collapsed-size="72"
      >
        <!-- Header Slot -->
        <template #header="{ collapsed }">
          <div class="flex items-center gap-3">
            <img
              src="https://vuejs.org/images/logo.png"
              alt="Vue Logo"
              class="h-8 w-8"
            />
            <span
              v-if="!collapsed"
              class="text-xl font-semibold whitespace-nowrap"
            >
              My App
            </span>
          </div>
        </template>

        <!-- Default Slot (Body) -->
        <template #default="{ collapsed }">
          <nav class="flex flex-col gap-1">
            <a
              v-for="item in navigation"
              :key="item.name"
              href="#"
              class="flex items-center gap-3 px-3 py-2 rounded-md text-sm font-medium whitespace-nowrap text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-800"
            >
              <UIcon :name="item.icon" class="h-5 w-5" />
              <span v-if="!collapsed">{{ item.name }}</span>
            </a>
          </nav>
        </template>

        <!-- Footer Slot -->
        <template #footer="{ collapsed }">
          <div class="flex items-center gap-3">
            <UAvatar
              src="https://randomuser.me/api/portraits/women/44.jpg"
              alt="User Avatar"
            />
            <div v-if="!collapsed" class="whitespace-nowrap">
              <p class="text-sm font-semibold">Jane Doe</p>
              <p class="text-xs text-gray-500 dark:text-gray-400">
                jane.doe@example.com
              </p>
            </div>
          </div>
        </template>
      </DashboardSidebar>

      <main class="flex-grow p-8 h-screen overflow-y-auto">
        <h1 class="text-2xl font-bold mb-4">Main Content Area</h1>
        <p class="mb-4">This area will take up the remaining space.</p>
        <UButton
          @click="isSidebarCollapsed = !isSidebarCollapsed"
          icon="i-heroicons-arrows-right-left"
          label="Toggle Sidebar"
        />
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import DashboardSidebar from '../components/DashboardSidebar.vue'; // Adjust path if needed

const isSidebarCollapsed = ref(false);

const navigation = [
  { name: 'Home', icon: 'i-heroicons-home' },
  { name: 'Dashboard', icon: 'i-heroicons-chart-bar' },
  { name: 'Settings', icon: 'i-heroicons-cog-6-tooth' },
];
</script>