<template>
  <q-layout view="hHh Lpr lFf">
    <q-header class="bg-transparent">
      <q-toolbar>
        <q-btn v-if="!$q.screen.xs && !$q.screen.sm" flat dense round :icon="leftDrawerOpen ? 'close' : 'menu'" aria-label="Menu" color="primary"
          @click="toggleLeftDrawer" />

        <q-toolbar-title class="text-primary">
          Lista de Compras
        </q-toolbar-title>

        <div class="text-primary" >
          <q-toggle v-model="dark" color="primary" icon="dark_mode" checked-icon="light_mode" />
        </div>
      </q-toolbar>
    </q-header>

    <q-drawer v-if="!$q.screen.xs && !$q.screen.sm" v-model="leftDrawerOpen" behavior="desktop" bordered>
      <q-list>
        <EssentialLink v-for="link in linksList" :key="link.title" v-bind="link" />
      </q-list>
    </q-drawer>

    <q-page-container style="width:100%;height:calc(100vh - 50px)" >
      <router-view />
    </q-page-container>
    <q-footer :class="$q.dark.isActive ? 'bg-grey-10 shadow-up-2' : 'bg-white shadow-up-2'">
      <q-tabs spread class="full-width" align="justify">
        <q-route-tab :icon="item.icon" :to="item.link" class="text-primary" color="primary" v-for="item in linksList" :key="item.title" />
      </q-tabs>
    </q-footer>
  </q-layout>
</template>

<script setup>
import { computed, ref } from 'vue'
import EssentialLink from 'components/EssentialLink.vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

const linksList = [
  {
    title: 'Lista',
    icon: 'list',
    link: '/'
  },
  {
    title: 'Informações',
    icon: 'info',
    link: '/info'
  },
  {
    title: 'Configurações',
    icon: 'settings',
    link: '/settings'
  }
]

const leftDrawerOpen = ref(false)
const dark = computed({
  get: () => $q.dark.isActive,
  set: (val) => {
    $q.dark.set(val)
  }
})
function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value
}
</script>
