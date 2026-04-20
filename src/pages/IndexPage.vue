<template>
  <q-page class="full-width full-height">
    <div class="flex items-center justify-between q-px-sm q-pt-xs q-pb-xs" style="height:32px">
      <span class="text-caption text-grey-6">{{ list.length }} {{ list.length === 1 ? 'item' : 'itens' }}</span>
      <q-btn flat dense round icon="inventory_2" color="info" @click="showArchived = true">
        <q-badge v-if="archiveList.length" color="info" floating>{{ archiveList.length }}</q-badge>
        <q-tooltip>Itens arquivados</q-tooltip>
      </q-btn>
    </div>

    <q-scroll-area class="full-width" style="height:calc(100% - 92px)" ref="scroll">
      <div style="height:50px;margin:5px 10px" v-for="(item, i) in list" :key="item.id">
        <q-input
          :ref="el => listRefs[i] = el"
          :for="item.id.toString()"
          v-model="item.title"
          outlined
          dense
          :rules="[val => !!val || 'Não deixe a descrição vazia']"
          :bg-color="`${error === item.id ? 'red-4' : 'transparent'}`"
          :input-style="{ padding: '10px', textDecoration: item.added ? 'line-through' : '' }"
        >
          <q-menu touch-position context-menu>
            <q-list>
              <q-item clickable dense @click="deleteItem(item, i)">
                <q-item-section side>
                  <q-icon color="negative" name="delete" />
                </q-item-section>
                <q-item-section class="text-uppercase text-xs text-negative">
                  Excluir Item
                </q-item-section>
              </q-item>
              <q-item clickable dense @click="archiveItem(item)">
                <q-item-section side>
                  <q-icon color="info" name="archive" />
                </q-item-section>
                <q-item-section class="text-uppercase text-xs text-info">
                  Arquivar Item
                </q-item-section>
              </q-item>
            </q-list>
          </q-menu>

          <template #append>
            <div class="flex flex-center" style="width:80px;">
              <q-btn @click="changeItem(item, 'sub')" icon="keyboard_arrow_left" dense color="primary" flat size="sm" />
              <input @keyup.enter="addItem" v-model="item.qtd" mask="#" inputmode="numeric" class="inputQtd" />
              <q-btn @click="changeItem(item, 'add')" icon="keyboard_arrow_right" dense color="primary" size="sm" flat />
            </div>
          </template>

          <template #prepend>
            <q-checkbox color="primary" unelevated dense size="lg" v-model="item.added" />
          </template>
        </q-input>
      </div>
    </q-scroll-area>

    <div class="full-width q-px-sm q-pb-sm" style="height:60px">
      <q-btn @click="addItem" icon="add" label="Adicionar item" outline color="primary" class="full-width" />
    </div>

    <!-- Archived Items Dialog -->
    <q-dialog v-model="showArchived" maximized transition-show="slide-up" transition-hide="slide-down">
      <q-card class="full-height column">
        <q-card-section class="row items-center bg-info text-white q-py-sm">
          <q-icon name="inventory_2" size="sm" class="q-mr-sm" />
          <div class="text-h6">Itens Arquivados</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>

        <div v-if="sortedArchiveList.length === 0" class="col flex flex-center column q-pa-xl">
          <q-icon name="inventory_2" size="5rem" color="grey-4" />
          <div class="text-grey-5 q-mt-md text-body1">Nenhum item arquivado</div>
          <div class="text-grey-4 q-mt-xs text-caption text-center">
            Clique com o botão direito em um item da lista para arquivá-lo
          </div>
        </div>

        <q-scroll-area class="col full-width" v-else>
          <q-list separator>
            <q-item v-for="item in sortedArchiveList" :key="`${item.id}_${item.archivedAt}`">
              <q-item-section>
                <q-item-label :style="{ textDecoration: item.added ? 'line-through' : 'none' }">
                  {{ item.title }}
                </q-item-label>
                <q-item-label caption>
                  Qtd: {{ item.qtd }} &bull; {{ formatDate(item.archivedAt) }}
                </q-item-label>
              </q-item-section>
              <q-item-section side>
                <div class="row no-wrap q-gutter-xs">
                  <q-btn flat dense round icon="unarchive" color="primary" @click="unarchiveItem(item)">
                    <q-tooltip>Desarquivar</q-tooltip>
                  </q-btn>
                  <q-btn flat dense round icon="delete_forever" color="negative" @click="deleteArchivedItem(item)">
                    <q-tooltip>Excluir permanentemente</q-tooltip>
                  </q-btn>
                </div>
              </q-item-section>
            </q-item>
          </q-list>
        </q-scroll-area>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup>
import { useStorage } from '@vueuse/core'
import { useQuasar } from 'quasar'
import { computed, nextTick, ref } from 'vue'
import moment from 'moment'

const showArchived = ref(false)
const scroll = ref(null)
const listRefs = ref([])
const q = useQuasar()
const error = ref(null)
const currentItem = ref({ id: null, title: null, qtd: 1, added: false })

const list = useStorage('list', [
  { id: 1, title: 'Açucar 5kg', qtd: 3, added: false }
])
const archiveList = useStorage('archiveList', [])

const sortedArchiveList = computed(() =>
  [...archiveList.value].sort((a, b) => new Date(b.archivedAt) - new Date(a.archivedAt))
)

function formatDate(dateStr) {
  return moment(dateStr).format('DD/MM/YYYY [às] HH:mm')
}

async function addItem() {
  const indexEmpty = list.value.findIndex(item => !item.title || !item.title.trim())

  if (indexEmpty !== -1) {
    await nextTick()
    const el = listRefs.value[indexEmpty]?.$el
    listRefs.value[indexEmpty]?.focus()
    if (el && scroll.value) {
      scroll.value.setScrollPosition('vertical', el.offsetTop, 300)
    }
    listRefs.value[indexEmpty]?.focus?.()
    return
  }

  currentItem.value.id = Date.now()
  list.value.push({ ...currentItem.value })

  await nextTick()

  const el = listRefs.value[listRefs.value.length - 1]?.$el
  if (el && scroll.value) {
    scroll.value.setScrollPosition('vertical', el.offsetTop, 300)
  }
  listRefs.value[listRefs.value.length - 1]?.focus?.()
  currentItem.value = { id: null, title: null, qtd: 1, added: false }
}

function changeItem(item, type) {
  if (type === 'add') item.qtd++
  if (type === 'sub' && item.qtd > 1) item.qtd--
}

function deleteItem(item, index) {
  error.value = item.id
  q.dialog({
    title: 'Atenção',
    message: 'Tem certeza que deseja excluir o item <b>' + item.title + '</b>?',
    html: true,
    ok: { color: 'negative', label: 'Excluir' },
    cancel: { outline: true, label: 'Cancelar', color: 'grey-8' }
  }).onOk(() => {
    list.value.splice(index, 1)
    error.value = null
  }).onDismiss(() => { error.value = null })
}

function archiveItem(item) {
  const index = list.value.findIndex(i => i.id === item.id)
  if (index === -1) return
  archiveList.value.push({ ...item, archivedAt: new Date().toISOString() })
  list.value.splice(index, 1)
}

function unarchiveItem(item) {
  const index = archiveList.value.findIndex(i => i.id === item.id && i.archivedAt === item.archivedAt)
  if (index === -1) return
  const { archivedAt, ...original } = archiveList.value[index]
  list.value.push(original)
  archiveList.value.splice(index, 1)
}

function deleteArchivedItem(item) {
  const index = archiveList.value.findIndex(i => i.id === item.id && i.archivedAt === item.archivedAt)
  if (index !== -1) archiveList.value.splice(index, 1)
}
</script>

<style>
.q-field--outlined .q-field__control {
  padding: 0 0 0 10px !important;
}

.inputQtd {
  width: 30px;
  text-align: center;
  font-weight: 900;
  color: var(--color-primary);
  font-size: 20px;
  border: none;
  background-color: transparent;
}

.inputQtd:focus {
  border: none;
  outline: none;
}
</style>
