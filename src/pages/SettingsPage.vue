<template>
  <q-page class="q-pa-md">
    <!-- PWA Install -->
    <q-list bordered separator class="rounded-borders q-mb-lg">
      <q-item-label header class="text-weight-bold">
        <q-icon name="install_mobile" class="q-mr-xs" />
        Instalar Aplicativo
      </q-item-label>

      <q-item v-if="isInstalled">
        <q-item-section avatar>
          <q-icon name="check_circle" color="positive" size="md" />
        </q-item-section>
        <q-item-section>
          <q-item-label>Aplicativo instalado</q-item-label>
          <q-item-label caption>O app já está instalado no seu dispositivo</q-item-label>
        </q-item-section>
      </q-item>

      <q-item v-else-if="isInstallable" class="q-py-md">
        <q-item-section>
          <q-btn
            @click="installApp"
            icon="install_mobile"
            label="Instalar aplicativo"
            color="primary"
            unelevated
            class="full-width"
            size="md"
          />
          <div class="text-caption text-grey-5 q-mt-sm text-center">
            Acesse mais rápido direto da sua tela inicial
          </div>
        </q-item-section>
      </q-item>

      <q-item v-else>
        <q-item-section avatar>
          <q-icon name="info" color="grey-5" size="md" />
        </q-item-section>
        <q-item-section>
          <q-item-label>Instalação não disponível</q-item-label>
          <q-item-label caption>
            Use um navegador compatível (Chrome, Edge) ou o app já está instalado
          </q-item-label>
        </q-item-section>
      </q-item>
    </q-list>

    <!-- Theme Color -->
    <q-list bordered separator class="rounded-borders">
      <q-item-label header class="text-weight-bold">
        <q-icon name="palette" class="q-mr-xs" />
        Tema
      </q-item-label>

      <q-item>
        <q-item-section>
          <q-item-label>Cor principal</q-item-label>
          <q-item-label caption>Escolha a cor do tema do aplicativo</q-item-label>
        </q-item-section>
        <q-item-section side>
          <q-btn
            round
            unelevated
            :style="{ backgroundColor: primaryColor }"
            size="md"
            @click="showColorPicker = true"
          >
            <q-tooltip>Alterar cor</q-tooltip>
          </q-btn>
        </q-item-section>
      </q-item>

      <q-item>
        <q-item-section>
          <q-item-label class="text-caption text-grey-6 q-mb-sm">Cores rápidas</q-item-label>
          <div class="row q-gutter-sm">
            <q-btn
              v-for="color in colorPresets"
              :key="color"
              round
              unelevated
              :style="{ backgroundColor: color }"
              size="sm"
              @click="applyColor(color)"
            >
              <q-icon v-if="primaryColor === color" name="check" color="white" size="xs" />
            </q-btn>
          </div>
        </q-item-section>
      </q-item>
    </q-list>

    <!-- Color Picker Dialog -->
    <q-dialog v-model="showColorPicker">
      <q-card style="min-width:320px">
        <q-card-section class="row items-center">
          <q-icon name="palette" color="primary" class="q-mr-sm" />
          <div class="text-h6">Escolher cor</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <q-color v-model="tempColor" no-footer default-view="palette" />
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Cancelar" v-close-popup />
          <q-btn
            flat
            label="Aplicar"
            color="primary"
            @click="applyColor(tempColor); showColorPicker = false"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import { setCssVar } from 'quasar'
import { useStorage } from '@vueuse/core'

const primaryColor = useStorage('primaryColor', '#e91e63')
const tempColor = ref(primaryColor.value)
const showColorPicker = ref(false)
const isInstalled = ref(false)
const isInstallable = ref(false)
const deferredPrompt = ref(null)

const colorPresets = [
  '#e91e63', '#9c27b0', '#673ab7', '#3f51b5',
  '#2196f3', '#009688', '#4caf50', '#ff5722',
  '#ff9800', '#795548', '#607d8b', '#212121'
]

onMounted(() => {
  setCssVar('primary', primaryColor.value)

  isInstalled.value =
    window.matchMedia('(display-mode: standalone)').matches ||
    !!window.navigator.standalone

  if (!isInstalled.value) {
    // Event may have already fired before this page mounted — check global capture
    if (window.__pwaPrompt) {
      deferredPrompt.value = window.__pwaPrompt
      isInstallable.value = true
    }
    // Also listen for future events
    window.addEventListener('beforeinstallprompt', handleInstallPrompt)
  }
})

onUnmounted(() => {
  window.removeEventListener('beforeinstallprompt', handleInstallPrompt)
})

function handleInstallPrompt(e) {
  e.preventDefault()
  window.__pwaPrompt = e
  deferredPrompt.value = e
  isInstallable.value = true
}

async function installApp() {
  if (!deferredPrompt.value) return
  deferredPrompt.value.prompt()
  const { outcome } = await deferredPrompt.value.userChoice
  if (outcome === 'accepted') {
    isInstalled.value = true
    isInstallable.value = false
  }
  deferredPrompt.value = null
  window.__pwaPrompt = null
}

function applyColor(color) {
  primaryColor.value = color
  tempColor.value = color
  setCssVar('primary', color)
}
</script>
