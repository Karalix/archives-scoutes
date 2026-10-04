<script setup lang="ts">
// 3.4 : pour une année protégée, seuls le libellé et le nombre de documents sont visibles
defineProps<{ label?: string, count?: number, notCovered?: boolean }>()
const { ask } = useUnlock()
</script>

<template>
  <div class="rounded-xl border border-dashed border-default p-8 text-center max-w-xl mx-auto">
    <UIcon name="i-lucide-lock" class="size-12 text-primary mx-auto" />
    <h2 class="mt-4 text-xl font-semibold">
      Archives récentes
    </h2>
    <p class="mt-2 text-muted">
      <template v-if="count !== undefined">
        {{ label }} contient {{ count }} document{{ count > 1 ? 's' : '' }}.
      </template>
      <template v-if="notCovered">
        Votre mot de passe ne couvre pas cette année : saisissez celui d'une année plus récente.
      </template>
      <template v-else>
        Pour protéger les jeunes, leurs images ne sont visibles qu'avec le mot de passe annuel transmis aux familles.
      </template>
    </p>
    <UButton class="mt-6" size="lg" icon="i-lucide-lock-open" @click="ask()">
      Saisir le mot de passe
    </UButton>
  </div>
</template>
