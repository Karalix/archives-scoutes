<script setup lang="ts">
const { data: site } = await useSite()
const { ask } = useUnlock()
const route = useRoute()
const toast = useToast()

async function lock() {
  await $fetch('/api/access/lock', { method: 'POST' })
  toast.add({ title: 'Archives récentes verrouillées sur cet appareil', icon: 'i-lucide-lock' })
  await refreshNuxtData()
}

const nav = computed(() => [
  { label: 'Années', to: '/', icon: 'i-lucide-calendar-range', active: route.path === '/' || route.path.startsWith('/annee') },
  { label: 'Recherche', to: '/recherche', icon: 'i-lucide-search' },
])
</script>

<template>
  <div class="min-h-dvh flex flex-col bg-default">
    <UHeader :to="'/'" :title="site?.name" mode="slideover">
      <template #title>
        <span class="flex items-center gap-2 min-w-0">
          <img v-if="site?.logoUrl" :src="site.logoUrl" alt="" class="h-8 w-8 rounded object-contain">
          <img v-else src="/icon.svg" alt="" class="h-8 w-8">
          <span class="truncate font-semibold">{{ site?.name }}</span>
        </span>
      </template>

      <UNavigationMenu :items="nav" variant="link" />

      <template #right>
        <UTooltip v-if="site?.family" :text="`Accès jusqu'à ${scoutYearLabel(site.family.maxYear)}`">
          <UButton color="neutral" variant="ghost" icon="i-lucide-lock-open" aria-label="Verrouiller les archives récentes" @click="lock" />
        </UTooltip>
        <UButton v-else color="neutral" variant="ghost" icon="i-lucide-lock" class="hidden sm:inline-flex" @click="ask()">
          Archives récentes
        </UButton>
        <UColorModeButton />
        <UButton v-if="site?.admin" to="/admin" color="neutral" variant="ghost" icon="i-lucide-settings" aria-label="Administration" />
      </template>

      <template #body>
        <UNavigationMenu :items="nav" orientation="vertical" class="-mx-2.5" />
        <USeparator class="my-4" />
        <UButton v-if="!site?.family" block icon="i-lucide-lock" @click="ask()">
          Saisir le mot de passe annuel
        </UButton>
        <UButton v-else block color="neutral" variant="outline" icon="i-lucide-lock-open" @click="lock">
          Verrouiller (accès jusqu'à {{ scoutYearLabel(site.family.maxYear) }})
        </UButton>
      </template>
    </UHeader>

    <UMain class="flex-1">
      <slot />
    </UMain>

    <USeparator />
    <UFooter>
      <template #left>
        <p class="text-sm text-muted">
          {{ site?.name }} · archives du groupe
        </p>
      </template>
      <template #right>
        <nav class="flex flex-wrap gap-x-4 gap-y-1 text-sm" aria-label="Pied de page">
          <NuxtLink to="/mentions-legales" class="text-muted hover:text-default">Mentions légales</NuxtLink>
          <NuxtLink to="/confidentialite" class="text-muted hover:text-default">Confidentialité</NuxtLink>
          <a v-if="site?.contactEmail" :href="`mailto:${site.contactEmail}`" class="text-muted hover:text-default">Contact</a>
          <NuxtLink to="/admin" class="text-muted hover:text-default">Administration</NuxtLink>
        </nav>
      </template>
    </UFooter>

    <PublicUnlockModal />
  </div>
</template>
