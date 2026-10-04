<script setup lang="ts">
// F-10 / F-11 : recherche plein texte et filtre par lieu, limitée aux documents accessibles
const route = useRoute()
const router = useRouter()
const { data: site } = await useSite()
const q = ref(String(route.query.q ?? ''))
const place = ref(String(route.query.lieu ?? ''))
const kind = ref(String(route.query.type ?? ''))
const branch = ref(String(route.query.branche ?? ''))

const query = computed(() => ({
  q: route.query.q || undefined,
  place: route.query.lieu || undefined,
  kind: route.query.type || undefined,
  branch: route.query.branche || undefined,
}))
const { data, pending } = await useFetch('/api/public/search', { query, key: 'search' })

let t: ReturnType<typeof setTimeout> | undefined
watch([q, place, kind, branch], () => {
  clearTimeout(t)
  t = setTimeout(() => {
    router.replace({ query: { q: q.value || undefined, lieu: place.value || undefined, type: kind.value || undefined, branche: branch.value || undefined } })
  }, 250)
})

const placeItems = computed(() => [{ label: 'Tous les lieux', value: '' }, ...(data.value?.places ?? []).map(p => ({ label: p, value: p }))])
const kindItems = [{ label: 'Tous les types', value: '' }, ...Object.entries(KIND_LABELS).map(([value, label]) => ({ label, value }))]
const branchItems = computed(() => [{ label: 'Toutes les branches', value: '' }, ...(site.value?.branches ?? []).map(b => ({ label: b.label, value: b.key }))])

useHead({ title: 'Recherche', meta: [{ name: 'robots', content: 'noindex' }] })
</script>

<template>
  <UContainer class="py-8 space-y-6">
    <h1 class="text-3xl font-bold">
      Recherche
    </h1>
    <div class="grid gap-3 sm:grid-cols-2 lg:grid-cols-4">
      <UInput v-model="q" icon="i-lucide-search" placeholder="Titre, lieu, description, mot-clé…" size="lg" class="sm:col-span-2 lg:col-span-1" aria-label="Rechercher" autofocus />
      <USelect v-model="place" :items="placeItems" size="lg" aria-label="Lieu de camp" />
      <USelect v-model="kind" :items="kindItems" size="lg" aria-label="Type" />
      <USelect v-model="branch" :items="branchItems" size="lg" aria-label="Branche" />
    </div>
    <p class="text-sm text-muted" aria-live="polite">
      <template v-if="pending">
        Recherche…
      </template>
      <template v-else>
        {{ data?.results.length ?? 0 }} résultat{{ (data?.results.length ?? 0) > 1 ? 's' : '' }}
        <template v-if="!site?.family">
          · les archives récentes n'apparaissent qu'avec le mot de passe annuel
        </template>
      </template>
    </p>
    <ul class="grid gap-4 grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
      <li v-for="d in data?.results ?? []" :key="d.id">
        <PublicDocCard :doc="d" show-year />
      </li>
    </ul>
  </UContainer>
</template>
