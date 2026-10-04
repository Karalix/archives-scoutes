<script setup lang="ts">
defineProps<{
  year: { startYear: number, label: string, count: number, locked: boolean, coverUrl: string | null }
}>()
</script>

<template>
  <NuxtLink
    :to="`/annee/${year.startYear}`"
    class="group block rounded-lg overflow-hidden ring ring-default bg-elevated/40 hover:ring-primary focus-visible:ring-primary transition"
    :aria-label="`${year.label}, ${year.count} documents${year.locked ? ', protégée par mot de passe' : ''}`"
  >
    <div class="relative aspect-video bg-accented">
      <img v-if="year.coverUrl" :src="year.coverUrl" alt="" loading="lazy" class="absolute inset-0 h-full w-full object-cover group-hover:scale-[1.02] transition-transform">
      <div v-else class="absolute inset-0 flex items-center justify-center text-dimmed">
        <UIcon :name="year.locked ? 'i-lucide-lock' : 'i-lucide-tent'" class="size-10" />
      </div>
      <UBadge v-if="year.locked" color="neutral" variant="solid" icon="i-lucide-lock" class="absolute top-2 right-2">
        Protégée
      </UBadge>
    </div>
    <div class="flex items-baseline justify-between gap-2 p-3">
      <span class="text-lg font-semibold tabular-nums">{{ year.label }}</span>
      <span class="text-sm text-muted">{{ year.count }} doc{{ year.count > 1 ? 's' : '' }}</span>
    </div>
  </NuxtLink>
</template>
