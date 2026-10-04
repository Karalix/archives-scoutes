<script setup lang="ts">
defineProps<{ title: string, description?: string, wide?: boolean }>()
const { data: site } = await useSite()
</script>

<template>
  <div class="flex min-h-dvh items-start justify-center bg-muted px-4 py-10 sm:items-center">
    <UCard class="w-full" :class="wide ? 'max-w-2xl' : 'max-w-md'">
      <template #header>
        <div class="flex items-center gap-3">
          <img v-if="site?.logoUrl" :src="site.logoUrl" alt="" class="size-10 rounded object-contain">
          <UIcon v-else name="i-lucide-tent-tree" class="size-10 text-primary" />
          <div class="min-w-0">
            <h1 class="text-lg font-semibold">
              {{ title }}
            </h1>
            <p v-if="description" class="text-sm text-muted">
              {{ description }}
            </p>
          </div>
        </div>
      </template>
      <slot />
      <template v-if="$slots.footer" #footer>
        <slot name="footer" />
      </template>
    </UCard>
  </div>
</template>
