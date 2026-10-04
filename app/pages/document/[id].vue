<script setup lang="ts">
const route = useRoute()
const { data: site } = await useSite()
const id = computed(() => String(route.params.id))
const { data: doc, error } = await useFetch(() => `/api/public/documents/${id.value}`, { key: `doc-${id.value}` })
const reportOpen = ref(false)

const status = computed(() => (error.value as any)?.statusCode as number | undefined)
const lockInfo = computed(() => (error.value as any)?.data as { reason?: string, yearStart?: number } | undefined)

// Photos voisines (balayage) : documents photo de la même année et du même événement
const { data: yearPage } = await useFetch(() => `/api/public/years/${doc.value?.yearStart}`, {
  key: `year-nav-${id.value}`,
  immediate: !!doc.value && doc.value.kind === 'photo',
  watch: false,
})
const siblings = computed(() => (yearPage.value?.documents ?? []).filter(d => d.kind === 'photo' && d.eventId === doc.value?.eventId))
const idx = computed(() => siblings.value.findIndex(d => d.id === id.value))
const prevId = computed(() => idx.value > 0 ? siblings.value[idx.value - 1]!.id : null)
const nextId = computed(() => idx.value >= 0 && idx.value < siblings.value.length - 1 ? siblings.value[idx.value + 1]!.id : null)

const dateLabel = computed(() => doc.value?.date ? new Date(`${doc.value.date}T12:00:00`).toLocaleDateString('fr-FR', { day: 'numeric', month: 'long', year: 'numeric' }) : '')

useHead(() => ({
  title: doc.value?.title ?? 'Document',
  meta: !doc.value || doc.value.protected ? [{ name: 'robots', content: 'noindex, nofollow' }] : [],
}))
useSeoMeta({
  ogTitle: () => doc.value && !doc.value.protected ? doc.value.title : undefined,
  description: () => doc.value && !doc.value.protected ? doc.value.description?.slice(0, 160) : undefined,
})
</script>

<template>
  <UContainer class="py-6 sm:py-8">
    <div v-if="status === 401" class="py-8">
      <PublicLockPanel :not-covered="lockInfo?.reason === 'yearNotCovered'" />
    </div>
    <div v-else-if="error" class="text-center py-16">
      <p class="text-lg">
        {{ apiError(error) }}
      </p>
      <UButton to="/" class="mt-4" variant="outline">
        Retour aux années
      </UButton>
    </div>

    <article v-else-if="doc" class="space-y-6">
      <UBreadcrumb
        :items="[
          { label: 'Années', to: '/' },
          { label: scoutYearLabel(doc.yearStart), to: `/annee/${doc.yearStart}` },
          ...(doc.event ? [{ label: doc.event.title }] : []),
        ]"
      />

      <div v-if="doc.kind === 'video'">
        <iframe
          v-if="doc.streamUrl"
          :src="doc.streamUrl"
          class="w-full aspect-video rounded-lg"
          allow="accelerometer; gyroscope; autoplay; encrypted-media; picture-in-picture; fullscreen"
          allowfullscreen
          :title="doc.title"
        />
        <PublicVideoPlayer
          v-else-if="doc.mainUrl"
          :id="doc.id"
          :src="doc.mainUrl"
          :poster="doc.thumbUrl"
          :captions="doc.captionsUrl"
          :chapters="doc.chapters"
          :can-download="doc.canDownload"
        />
      </div>
      <PublicPhotoViewer v-else-if="doc.kind === 'photo' && doc.mainUrl" :src="doc.mainUrl" :alt="doc.title" :prev-id="prevId" :next-id="nextId" />
      <ClientOnly v-else-if="doc.kind === 'pdf' && doc.mainUrl">
        <PublicPdfViewer :src="doc.mainUrl" />
      </ClientOnly>
      <div v-else-if="doc.kind === 'audio' && doc.mainUrl" class="rounded-lg bg-elevated p-6 flex flex-col sm:flex-row gap-4 items-center">
        <img v-if="doc.thumbUrl" :src="doc.thumbUrl" alt="" class="size-32 rounded object-cover">
        <UIcon v-else name="i-lucide-music" class="size-16 text-primary" />
        <audio :src="doc.mainUrl" controls preload="metadata" class="w-full" :controlslist="doc.canDownload ? undefined : 'nodownload'" />
      </div>
      <p v-else class="text-muted">
        Fichier en cours de préparation.
      </p>

      <div class="grid gap-6 lg:grid-cols-3">
        <div class="lg:col-span-2 space-y-3">
          <h1 class="text-2xl sm:text-3xl font-bold">
            {{ doc.title }}
          </h1>
          <p v-if="doc.description" class="whitespace-pre-line text-muted">
            {{ doc.description }}
          </p>
          <div v-if="doc.tags.length" class="flex flex-wrap gap-2">
            <UBadge v-for="t in doc.tags" :key="t" color="neutral" variant="subtle">
              {{ t }}
            </UBadge>
          </div>
        </div>
        <aside class="space-y-4">
          <!-- F-09 : métadonnées -->
          <dl class="grid grid-cols-[auto_1fr] gap-x-4 gap-y-2 text-sm">
            <dt class="text-muted">
              Année
            </dt>
            <dd><NuxtLink :to="`/annee/${doc.yearStart}`" class="underline">{{ scoutYearLabel(doc.yearStart) }}</NuxtLink></dd>
            <template v-if="doc.event">
              <dt class="text-muted">
                Événement
              </dt>
              <dd>{{ doc.event.title }}</dd>
            </template>
            <template v-if="dateLabel">
              <dt class="text-muted">
                Date
              </dt>
              <dd>{{ dateLabel }}</dd>
            </template>
            <template v-if="doc.place">
              <dt class="text-muted">
                Lieu
              </dt>
              <dd><NuxtLink :to="{ path: '/recherche', query: { lieu: doc.place } }" class="underline">{{ doc.place }}</NuxtLink></dd>
            </template>
            <template v-if="doc.branch">
              <dt class="text-muted">
                Branche
              </dt>
              <dd>{{ branchLabel(site, doc.branch) }}</dd>
            </template>
            <template v-if="doc.duration">
              <dt class="text-muted">
                Durée
              </dt>
              <dd class="tabular-nums">
                {{ formatDuration(doc.duration) }}
              </dd>
            </template>
            <template v-if="doc.credits">
              <dt class="text-muted">
                Crédits
              </dt>
              <dd>{{ doc.credits }}</dd>
            </template>
            <template v-if="doc.people">
              <dt class="text-muted">
                Personnes
              </dt>
              <dd>{{ doc.people }}</dd>
            </template>
          </dl>
          <div class="flex flex-wrap gap-2">
            <UButton v-if="doc.downloadUrl" :to="doc.downloadUrl" external icon="i-lucide-download" color="neutral" variant="outline">
              Télécharger
            </UButton>
            <UButton icon="i-lucide-flag" color="neutral" variant="ghost" @click="reportOpen = true">
              Signaler / demander un retrait
            </UButton>
            <UButton v-if="doc.isAdmin" :to="`/admin/documents/${doc.id}`" icon="i-lucide-pencil" color="neutral" variant="ghost">
              Modifier
            </UButton>
          </div>
          <p v-if="doc.protected" class="text-xs text-muted flex gap-1.5">
            <UIcon name="i-lucide-lock" class="size-3.5 mt-0.5 shrink-0" />
            Archive récente : consultation en ligne uniquement, merci de ne pas la diffuser.
          </p>
        </aside>
      </div>
      <PublicReportModal v-model:open="reportOpen" :document-id="doc.id" :delay-days="site?.takedownDelayDays" />
    </article>
  </UContainer>
</template>
