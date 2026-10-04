<script setup lang="ts">
const { data: site } = await useSite()
const { data: years } = await useFetch('/api/public/years', { key: 'years' })

// F-03 : regroupement par décennie au-delà de ~20 années
const groups = computed(() => {
  const list = years.value ?? []
  if (list.length <= 20) return [{ label: null as string | null, years: list }]
  const map = new Map<number, typeof list>()
  for (const y of list) {
    const d = Math.floor(y.startYear / 10) * 10
    if (!map.has(d)) map.set(d, [])
    map.get(d)!.push(y)
  }
  return [...map.entries()].map(([d, ys]) => ({ label: `Années ${d}`, years: ys }))
})

useSeoMeta({ description: () => site.value?.intro?.slice(0, 160) || `Archives de ${site.value?.name}` })
</script>

<template>
  <UContainer class="py-8 sm:py-12 space-y-10">
    <section class="max-w-3xl">
      <h1 class="text-3xl sm:text-4xl font-bold tracking-tight">
        {{ site?.name }}
      </h1>
      <!-- eslint-disable-next-line vue/no-v-html -- miniMarkdown échappe tout le HTML -->
      <div v-if="site?.intro" class="prose-lite mt-4 text-lg text-muted" v-html="miniMarkdown(site.intro)" />
      <p v-else class="mt-4 text-lg text-muted">
        Photos, montages vidéo de camps, chants, carnets et journaux du groupe, classés par année scoute.
      </p>
    </section>

    <section v-if="!years?.length" class="text-center py-16 text-muted">
      <UIcon name="i-lucide-archive" class="size-12 mx-auto mb-3" />
      <p>Aucune archive publiée pour l'instant.</p>
    </section>

    <section v-for="g in groups" :key="g.label ?? 'all'" :aria-labelledby="g.label ? `d-${g.label}` : undefined">
      <h2 v-if="g.label" :id="`d-${g.label}`" class="text-xl font-semibold mb-4">
        {{ g.label }}
      </h2>
      <h2 v-else class="sr-only">
        Frise des années
      </h2>
      <ul class="grid gap-4 grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 2xl:grid-cols-4">
        <li v-for="y in g.years" :key="y.startYear">
          <PublicYearCard :year="y" />
        </li>
      </ul>
    </section>

    <p v-if="years?.some(y => y.locked)" class="text-sm text-muted flex items-start gap-2">
      <UIcon name="i-lucide-shield" class="size-4 mt-0.5 shrink-0" />
      Pour protéger les jeunes, les années récentes ne sont visibles qu'avec le mot de passe annuel transmis aux familles.
    </p>
  </UContainer>
</template>
