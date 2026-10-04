<script setup lang="ts">
// A-15 : tableau de bord ; R-08 : rappels ; D-02 : mises à jour ; V-05 : copie hors ligne
definePageMeta({ layout: 'admin', middleware: 'admin' })
useHead({ title: 'Tableau de bord' })

interface Dashboard {
  user: { name: string, role: AdminRole }
  currentYear: number
  currentYearLabel: string
  pivot: number
  pivotMode: 'fixed' | 'sliding'
  years: number
  documents: Partial<Record<'draft' | 'published' | 'trashed', number>>
  openReports: number
  storage: { used: number, quota: number, videoMinutes: number }
  month: { bytesServed: number, views: number }
  cost: { r2: number, stream: number, total: number, currency: string }
  reminders: string[]
  version: string
  latestVersion: string | null
  storageDriver: string
  keepOriginals: boolean
}

const { can } = useAdmin()
const NuxtLinkC = resolveComponent('NuxtLink')
const { data, pending, refresh } = await useFetch<Dashboard>('/api/admin/dashboard', { key: 'admin-dashboard' })

const storagePct = computed(() => data.value?.storage.quota ? Math.min(100, Math.round(data.value.storage.used / data.value.storage.quota * 100)) : 0)

function newer(a: string | null | undefined, b: string | null | undefined) {
  if (!a || !b) return false
  const pa = a.split('.').map(Number)
  const pb = b.split('.').map(Number)
  for (let i = 0; i < 3; i++) {
    if ((pa[i] ?? 0) !== (pb[i] ?? 0)) return (pa[i] ?? 0) > (pb[i] ?? 0)
  }
  return false
}
const updateAvailable = computed(() => newer(data.value?.latestVersion, data.value?.version))

const reminderLink = (r: string) => {
  if (r.includes('mot de passe')) return can('editor') ? '/admin/mots-de-passe' : undefined
  if (r.includes('pivot')) return can('editor') ? '/admin/parametres' : undefined
  if (r.includes('propriétaire')) return can('owner') ? '/admin/administrateurs' : undefined
  return undefined
}
</script>

<template>
  <AdminPage title="Tableau de bord">
    <template #actions>
      <UButton icon="i-lucide-refresh-cw" color="neutral" variant="ghost" aria-label="Actualiser" :loading="pending" @click="() => refresh()" />
      <UButton to="/admin/televerser" icon="i-lucide-upload" label="Téléverser" class="hidden sm:inline-flex" />
    </template>

    <div v-if="!data" class="grid gap-4 sm:grid-cols-2 lg:grid-cols-4">
      <USkeleton v-for="i in 4" :key="i" class="h-24" />
    </div>

    <template v-else>
      <div v-if="data.reminders.length" class="space-y-2">
        <UAlert
          v-for="r in data.reminders"
          :key="r"
          color="warning"
          variant="subtle"
          icon="i-lucide-bell-ring"
          :title="r"
          :actions="reminderLink(r) ? [{ label: 'Y aller', to: reminderLink(r), color: 'warning', variant: 'outline' }] : undefined"
        />
      </div>

      <div class="grid gap-4 sm:grid-cols-2 lg:grid-cols-4">
        <UCard>
          <p class="text-sm text-muted">
            Documents publiés
          </p>
          <p class="text-3xl font-semibold">
            {{ data.documents.published ?? 0 }}
          </p>
        </UCard>
        <NuxtLink to="/admin/documents?status=draft" class="block rounded-lg focus-visible:outline-2">
          <UCard class="h-full hover:bg-elevated/50">
            <p class="text-sm text-muted">
              Brouillons à relire
            </p>
            <p class="text-3xl font-semibold" :class="data.documents.draft ? 'text-warning' : ''">
              {{ data.documents.draft ?? 0 }}
            </p>
          </UCard>
        </NuxtLink>
        <NuxtLink to="/admin/annees" class="block rounded-lg">
          <UCard class="h-full hover:bg-elevated/50">
            <p class="text-sm text-muted">
              Années
            </p>
            <p class="text-3xl font-semibold">
              {{ data.years }}
            </p>
          </UCard>
        </NuxtLink>
        <component :is="can('editor') ? NuxtLinkC : 'div'" to="/admin/signalements" class="block rounded-lg">
          <UCard class="h-full" :class="can('editor') ? 'hover:bg-elevated/50' : ''">
            <p class="text-sm text-muted">
              Signalements ouverts
            </p>
            <p class="text-3xl font-semibold" :class="data.openReports ? 'text-error' : ''">
              {{ data.openReports }}
            </p>
          </UCard>
        </component>
      </div>

      <div class="grid gap-4 lg:grid-cols-2">
        <UCard>
          <template #header>
            <h2 class="flex items-center gap-2 font-semibold">
              <UIcon name="i-lucide-hard-drive" /> Stockage
            </h2>
          </template>
          <div class="space-y-3">
            <div class="flex items-baseline justify-between gap-2">
              <span class="text-2xl font-semibold">{{ formatBytes(data.storage.used) }}</span>
              <span class="text-sm text-muted">{{ data.storage.quota ? `sur ${formatBytes(data.storage.quota)} (${storagePct} %)` : 'sans quota' }}</span>
            </div>
            <UProgress v-if="data.storage.quota" :model-value="storagePct" :color="storagePct >= 80 ? 'error' : 'primary'" />
            <UAlert v-if="storagePct >= 80" color="error" variant="subtle" icon="i-lucide-triangle-alert" title="Plus de 80 % du quota utilisé (S-04)" description="Un propriétaire peut relever le quota dans les paramètres ou faire du tri dans la corbeille." />
            <dl class="grid grid-cols-2 gap-3 text-sm">
              <div>
                <dt class="text-muted">
                  Vidéo encodée
                </dt>
                <dd class="font-medium">
                  {{ data.storage.videoMinutes.toLocaleString('fr-FR') }} min
                </dd>
              </div>
              <div>
                <dt class="text-muted">
                  Originaux conservés
                </dt>
                <dd class="font-medium">
                  {{ data.keepOriginals ? 'Oui' : 'Non' }}
                </dd>
              </div>
              <div>
                <dt class="text-muted">
                  Diffusé ce mois-ci
                </dt>
                <dd class="font-medium">
                  {{ formatBytes(data.month.bytesServed) }}
                </dd>
              </div>
              <div>
                <dt class="text-muted">
                  Consultations ce mois-ci
                </dt>
                <dd class="font-medium">
                  {{ data.month.views.toLocaleString('fr-FR') }}
                </dd>
              </div>
            </dl>
          </div>
        </UCard>

        <UCard>
          <template #header>
            <h2 class="flex items-center gap-2 font-semibold">
              <UIcon name="i-lucide-receipt" /> Coût estimé (mensuel)
            </h2>
          </template>
          <div class="space-y-3">
            <p class="text-2xl font-semibold">
              {{ data.cost.total.toLocaleString('fr-FR', { style: 'currency', currency: data.cost.currency }) }}
            </p>
            <dl class="grid grid-cols-2 gap-3 text-sm">
              <div>
                <dt class="text-muted">
                  Stockage R2
                </dt>
                <dd class="font-medium">
                  {{ data.cost.r2.toLocaleString('fr-FR', { style: 'currency', currency: data.cost.currency }) }}
                </dd>
              </div>
              <div>
                <dt class="text-muted">
                  Cloudflare Stream
                </dt>
                <dd class="font-medium">
                  {{ data.cost.stream ? data.cost.stream.toLocaleString('fr-FR', { style: 'currency', currency: data.cost.currency }) : 'désactivé' }}
                </dd>
              </div>
            </dl>
            <p class="text-xs text-muted">
              Estimation hors TVA : 10 Go gratuits puis 0,015 $/Go sur R2, sortie de données gratuite. Workers et D1 restent dans l'offre gratuite pour un groupe type (D-04).
            </p>
          </div>
        </UCard>

        <UCard>
          <template #header>
            <h2 class="flex items-center gap-2 font-semibold">
              <UIcon name="i-lucide-lock" /> Accès
            </h2>
          </template>
          <div class="space-y-2 text-sm">
            <p>Année scoute en cours : <strong>{{ data.currentYearLabel }}</strong></p>
            <p>
              Année pivot : <strong>{{ scoutYearLabel(data.pivot) }}</strong>
              <span class="text-muted">({{ data.pivotMode === 'sliding' ? 'mode glissant' : 'mode fixe' }})</span>
            </p>
            <p class="text-muted">
              Les années jusqu'à {{ scoutYearLabel(data.pivot) }} sont publiques ; les suivantes sont réservées aux familles avec le mot de passe annuel.
            </p>
            <div class="flex flex-wrap gap-2 pt-1">
              <UButton v-if="can('editor')" to="/admin/mots-de-passe" label="Mots de passe" icon="i-lucide-key-round" size="sm" variant="outline" />
              <UButton v-if="can('editor')" to="/admin/parametres" label="Régler le pivot" icon="i-lucide-settings" size="sm" variant="ghost" />
            </div>
          </div>
        </UCard>

        <UCard>
          <template #header>
            <h2 class="flex items-center gap-2 font-semibold">
              <UIcon name="i-lucide-package" /> Version
            </h2>
          </template>
          <div class="space-y-3 text-sm">
            <p>
              Version installée : <strong>{{ data.version }}</strong>
              <UBadge v-if="!updateAvailable && data.latestVersion" color="success" variant="subtle" class="ms-2">
                à jour
              </UBadge>
            </p>
            <UAlert
              v-if="updateAvailable"
              color="info"
              icon="i-lucide-sparkles"
              :title="`Mise à jour disponible : ${data.latestVersion}`"
              description="Chaque semaine, une GitHub Action du dépôt du groupe propose une pull request de synchronisation avec le dépôt source. Fusionnez-la sur GitHub : le site se redéploie et la base est migrée automatiquement (D-02)."
            />
            <p class="text-xs text-muted">
              Stockage : {{ data.storageDriver }}
            </p>
          </div>
        </UCard>
      </div>

      <UAlert
        color="neutral"
        variant="outline"
        icon="i-lucide-hard-drive-download"
        title="Gardez une copie des originaux hors ligne (V-05)"
        description="L'application n'est pas un système de sauvegarde : conservez aussi les fichiers originaux sur un disque du groupe, et faites régulièrement un export complet."
      />
    </template>
  </AdminPage>
</template>
