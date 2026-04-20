<template>
  <router-view />
</template>

<script setup>
import { onMounted } from 'vue'
import { setCssVar } from 'quasar'

onMounted(() => {
  const saved = localStorage.getItem('primaryColor')
  if (saved) setCssVar('primary', saved)
})
</script>
