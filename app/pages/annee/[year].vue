<script setup lang="ts">
const route = useRoute()
const { data: site } = await useSite()
const year = computed(() => Number(route.params.year))
const { data: page, error } = await useFetch(() => `/api/public/years/${year.value}`, { key: `year-${year.value}` })

const kind = ref<string>('all')
const branch = ref<string>('all')

const kindItems = computed(() => {
  const kinds = [...new Set(page.value?.documents.map(d => d.kind) ?? [])]
  return [{ label: 'Tous les types', value: 'all' }, ...kinds.map(k => ({ label: KIND_LABELS[k] ?? k, value: k }))]
})
const branchItems = computed(() => {
  const used = new Set(page.value?.documents.map(d => d.branch).filter(Boolean))
  return [{ label: 'Toutes les branches', value: 'all' }, ...(site.value?.branches ?? []).filter(b => used.has(b.key)).map(b => ({ label: b.label, value: b.key }))]
})

const filtered = computed(() => (page.value?.documents ?? []).filter(d =>
  (kind.value === 'all' || d.kind === kind.value) && (branch.value === 'all' || d.branch === branch.value)))

// F-04 : par événement, puis par unité/branche
const sections = computed(() => {
  const events = page.value?.events ?? []
  const order = site.value?.branches.map(b => b.key) ?? []
  const groupByBranch = (docs: typeof filtered.value) => {
    const m = new Map<string, typeof docs>()
    for (const d of docs) {
      const k = d.branch ?? ''
      if (!m.has(k)) m.set(k, [])
      m.get(k)!.push(d)
    }
    return [...m.entries()]
      .sort((a, b) => (a[0] ? order.indexOf(a[0]) : 99) - (b[0] ? order.indexOf(b[0]) : 99))
      .map(([key, docs]) => ({ key, label: key ? branchLabel(site.value, key) : '', docs }))
  }
  const out = events.map(e => ({ id: e.id, event: e, branches: groupByBranch(filtered.value.filter(d => d.eventId === e.id)) }))
  const loose = filtered.value.filter(d => !d.eventId || !events.some(e => e.id === d.eventId))
  if (loose.length) out.push({ id: 'autres', event: null as any, branches: groupByBranch(loose) })
  return out.filter(s => s.branches.length)
})

const hasVideos = computed(() => page.value?.documents.some(d => d.kind === 'video'))

useHead(() => ({
  title: page.value?.label ?? scoutYearLabel(year.value),
  meta: page.value && !page.value.public ? [{ name: 'robots', content: 'noindex, nofollow' }] : [],
}))

function eventDates(e: { startDate: string | null, endDate: string | null }) {
  const f = (s: string) => new Date(`${s}T12:00:00`).toLocaleDateString('fr-FR', { day: 'numeric', month: 'long' })
  if (e.startDate && e.endDate && e.endDate !== e.startDate) return `du ${f(e.startDate)} au ${f(e.endDate)}`
  return e.startDate ? f(e.startDate) : ''
}
</script>

<template>
  <UContainer class="py-8 space-y-8">
    <div v-if="error" class="text-center py-16">
      <p class="text-lg">
        {{ apiError(error) }}
      </p>
      <UButton to="/" class="mt-4" variant="outline">
        Retour aux années
      </UButton>
    </div>

    <template v-else-if="page">
      <header class="flex flex-wrap items-center gap-4 justify-between">
        <div>
          <p class="text-sm text-muted">
            Année scoute
          </p>
          <h1 class="text-3xl sm:text-4xl font-bold tabular-nums flex items-center gap-3">
            {{ page.label }}
            <UBadge v-if="!page.public" color="neutral" variant="subtle" icon="i-lucide-lock">
              Protégée
            </UBadge>
          </h1>
        </div>
        <nav class="flex items-center gap-2" aria-label="Années voisines">
          <UButton v-if="page.prev !== null" :to="`/annee/${page.prev}`" color="neutral" variant="outline" icon="i-lucide-chevron-left">
            {{ scoutYearLabel(page.prev) }}
          </UButton>
          <UButton v-if="page.next !== null" :to="`/annee/${page.next}`" color="neutral" variant="outline" trailing-icon="i-lucide-chevron-right">
            {{ scoutYearLabel(page.next) }}
          </UButton>
        </nav>
      </header>

      <p v-if="page.description" class="max-w-3xl text-muted whitespace-pre-line">
        {{ page.description }}
      </p>

      <PublicLockPanel v-if="page.locked && !page.documents.length" :label="page.label" :count="page.count" :not-covered="!!site?.family" />

      <template v-else>
        <div class="flex flex-wrap gap-3 items-center">
          <USelect v-model="kind" :items="kindItems" class="w-44" aria-label="Filtrer par type" />
          <USelect v-model="branch" :items="branchItems" class="w-56" aria-label="Filtrer par branche" />
          <UButton v-if="hasVideos" :to="`/soiree/${page.startYear}`" color="neutral" variant="soft" icon="i-lucide-tv" class="ml-auto">
            Mode soirée
          </UButton>
        </div>

        <p v-if="page.hiddenCount" class="text-sm text-muted flex items-center gap-2">
          <UIcon name="i-lucide-lock" class="size-4" />
          {{ page.hiddenCount }} document{{ page.hiddenCount > 1 ? 's' : '' }} protégé{{ page.hiddenCount > 1 ? 's' : '' }} par mot de passe.
        </p>

        <section v-for="s in sections" :key="s.id" class="space-y-4">
          <div v-if="s.event" class="border-b border-default pb-2">
            <h2 class="text-2xl font-semibold">
              {{ s.event.title }}
            </h2>
            <p class="text-sm text-muted">
              {{ [s.event.type !== s.event.title ? s.event.type : '', s.event.place, eventDates(s.event)].filter(Boolean).join(' · ') }}
            </p>
          </div>
          <h2 v-else-if="sections.length > 1" class="text-2xl font-semibold border-b border-default pb-2">
            Autres documents
          </h2>
          <div v-for="b in s.branches" :key="b.key" class="space-y-3">
            <h3 v-if="b.label && (s.branches.length > 1 || b.label)" class="text-base font-medium text-muted">
              {{ b.label }}
            </h3>
            <ul class="grid gap-4 grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 3xl:grid-cols-5">
              <li v-for="d in b.docs" :key="d.id">
                <PublicDocCard :doc="d" />
              </li>
            </ul>
          </div>
        </section>
        <p v-if="!sections.length" class="text-muted">
          Aucun document ne correspond à ces filtres.
        </p>
      </template>
    </template>
  </UContainer>
</template>
