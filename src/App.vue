<template>
  <router-view />
</template>

<script setup>
import { onMounted } from 'vue'
import { setCssVar } from 'quasar'

// Capture PWA prompt as early as possible — before any page component mounts.
// SettingsPage reads window.__pwaPrompt on mount to show the install button.
window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault()
  window.__pwaPrompt = e
})

onMounted(() => {
  const saved = localStorage.getItem('primaryColor')
  if (saved) setCssVar('primary', saved)
})
</script>
