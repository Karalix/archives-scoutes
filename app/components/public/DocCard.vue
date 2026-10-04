<script setup lang="ts">
const props = defineProps<{
  doc: { id: string, kind: string, title: string, thumbUrl: string | null, duration: number | null, branch: string | null, place?: string, date?: string | null, yearStart?: number }
  showYear?: boolean
}>()
const { data: site } = useSite()
const branch = computed(() => site.value?.branches.find(b => b.key === props.doc.branch))
</script>

<template>
  <NuxtLink :to="`/document/${doc.id}`" class="group block rounded-lg overflow-hidden ring ring-default hover:ring-primary transition bg-elevated/30">
    <div class="relative aspect-video bg-accented">
      <img v-if="doc.thumbUrl" :src="doc.thumbUrl" alt="" loading="lazy" class="absolute inset-0 h-full w-full object-cover">
      <div v-else class="absolute inset-0 flex items-center justify-center text-dimmed">
        <UIcon :name="KIND_ICONS[doc.kind] ?? 'i-lucide-file'" class="size-10" />
      </div>
      <span v-if="doc.kind === 'video'" class="absolute inset-0 flex items-center justify-center">
        <span class="rounded-full bg-black/55 p-3 text-white group-hover:bg-primary transition">
          <UIcon name="i-lucide-play" class="size-6" />
        </span>
      </span>
      <span v-if="doc.duration" class="absolute bottom-2 right-2 rounded bg-black/70 px-1.5 py-0.5 text-xs text-white tabular-nums">
        {{ formatDuration(doc.duration) }}
      </span>
    </div>
    <div class="p-3 space-y-1">
      <p class="font-medium leading-snug line-clamp-2">
        {{ doc.title }}
      </p>
      <p class="flex flex-wrap items-center gap-x-2 gap-y-1 text-xs text-muted">
        <span class="inline-flex items-center gap-1">
          <UIcon :name="KIND_ICONS[doc.kind] ?? 'i-lucide-file'" class="size-3.5" />{{ KIND_LABELS[doc.kind] }}
        </span>
        <span v-if="branch" class="inline-flex items-center gap-1">
          <span class="size-2 rounded-full" :style="{ background: branch.color ?? 'currentColor' }" />{{ branch.label }}
        </span>
        <span v-if="showYear && doc.yearStart">{{ scoutYearLabel(doc.yearStart) }}</span>
        <span v-if="doc.place">{{ doc.place }}</span>
      </p>
    </div>
  </NuxtLink>
</template>
