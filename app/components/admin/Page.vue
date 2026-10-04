<script setup lang="ts">
defineProps<{ title: string, id?: string }>()
</script>

<template>
  <UDashboardPanel :id="id ?? 'admin-page'">
    <template #header>
      <UDashboardNavbar :title="title">
        <template #leading>
          <UDashboardSidebarCollapse />
        </template>
        <template #right>
          <slot name="actions" />
        </template>
      </UDashboardNavbar>
      <UDashboardToolbar v-if="$slots.toolbar">
        <slot name="toolbar" />
      </UDashboardToolbar>
    </template>
    <template #body>
      <div class="mx-auto w-full max-w-6xl space-y-6">
        <slot />
      </div>
    </template>
  </UDashboardPanel>
</template>
