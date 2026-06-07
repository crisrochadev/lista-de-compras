<template>
  <q-page class="full-width full-height">
    <div class="flex items-center justify-between q-px-sm q-pt-xs q-pb-xs" style="height:32px">
      <span class="text-caption text-grey-6">{{ list.length }} {{ list.length === 1 ? 'item' : 'itens' }}</span>
      <div class="row no-wrap items-center q-gutter-xs">
        <q-btn flat dense round icon="create_new_folder" color="primary" @click="addCategory">
          <q-tooltip>Adicionar categoria</q-tooltip>
        </q-btn>
        <q-btn flat dense round icon="inventory_2" color="info" @click="showArchived = true">
          <q-badge v-if="archiveList.length" color="info" floating>{{ archiveList.length }}</q-badge>
          <q-tooltip>Itens arquivados</q-tooltip>
        </q-btn>
      </div>
    </div>

    <q-scroll-area class="full-width" style="height:calc(100% - 92px)" ref="scroll">
      <div class="q-px-sm q-pb-sm">
        <q-card
          v-for="(category, categoryIndex) in categories"
          :key="category.id"
          class="category-card q-mb-sm"
          :class="{ 'dragging-card': dragState?.type === 'category' && dragState.categoryId === category.id }"
          :data-category-id="category.id"
        >
          <q-card-section class="category-header row items-center no-wrap q-pa-sm">
            <q-icon
              name="drag_indicator"
              size="sm"
              color="grey-7"
              class="drag-handle q-mr-xs"
              @pointerdown="startCategoryDrag($event, category.id)"
            />
            <q-input
              :ref="el => categoryRefs[category.id] = el"
              v-model="category.title"
              borderless
              dense
              class="category-title col"
              input-class="text-subtitle1 text-weight-bold"
              placeholder="Nome da categoria"
              :rules="[val => !!val || 'Informe a categoria']"
            />
            <q-btn
              v-if="categories.length > 1"
              flat
              dense
              round
              icon="delete"
              color="negative"
              @click="deleteCategory(category)"
            >
              <q-tooltip>Excluir categoria</q-tooltip>
            </q-btn>
          </q-card-section>

          <q-card-section class="q-pa-sm q-pt-none">
            <div
              v-if="category.items.length === 0"
              class="empty-category text-caption text-grey-5 text-center q-pa-md"
              :data-category-id="category.id"
            >
              Arraste itens para esta categoria
            </div>

            <div
              v-for="(item, itemIndex) in category.items"
              :key="item.id"
              class="item-row q-mb-xs"
              :class="{ 'dragging-item': dragState?.type === 'item' && dragState.itemId === item.id }"
              :data-category-id="category.id"
              :data-item-id="item.id"
            >
              <q-input
                :ref="el => setItemRef(item.id, el)"
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
                    <q-item clickable dense @click="deleteItem(item, categoryIndex, itemIndex)">
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
                  <div class="row no-wrap items-center">
                    <q-icon
                      name="drag_indicator"
                      size="sm"
                      color="grey-7"
                      class="drag-handle item-drag-handle q-mr-xs"
                      @pointerdown="startItemDrag($event, category.id, item.id)"
                    />
                    <q-checkbox color="primary" unelevated dense size="lg" v-model="item.added" />
                  </div>
                </template>
              </q-input>
            </div>
          </q-card-section>
        </q-card>
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
                  Qtd: {{ item.qtd }} &bull; {{ item.categoryTitle || 'Sem categoria' }} &bull; {{ formatDate(item.archivedAt) }}
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
import { computed, nextTick, onBeforeUnmount, ref } from 'vue'
import moment from 'moment'

const defaultItems = [
  { id: 1, title: 'Açucar 5kg', qtd: 3, added: false }
]

const showArchived = ref(false)
const scroll = ref(null)
const listRefs = ref({})
const categoryRefs = ref({})
const q = useQuasar()
const error = ref(null)
const currentItem = ref({ id: null, title: null, qtd: 1, added: false })
const dragState = ref(null)

const legacyList = useStorage('list', defaultItems)
const categories = useStorage('categories', [
  { id: createId(), title: 'Mercado', items: legacyList.value.length ? legacyList.value : defaultItems }
])
const archiveList = useStorage('archiveList', [])

const list = computed(() => categories.value.flatMap(category => category.items))
const sortedArchiveList = computed(() =>
  [...archiveList.value].sort((a, b) => new Date(b.archivedAt) - new Date(a.archivedAt))
)

function formatDate(dateStr) {
  return moment(dateStr).format('DD/MM/YYYY [às] HH:mm')
}

function setItemRef(id, el) {
  if (el) listRefs.value[id] = el
}

async function addItem() {
  const empty = findEmptyItem()

  if (empty) {
    await focusItem(empty.item.id)
    return
  }

  const targetCategory = categories.value.at(-1) || createCategory('Mercado')
  const newItem = { ...currentItem.value, id: createId() }
  targetCategory.items.push(newItem)

  await nextTick()
  await focusItem(newItem.id)
  currentItem.value = { id: null, title: null, qtd: 1, added: false }
}

async function addCategory() {
  const category = createCategory(`Categoria ${categories.value.length + 1}`)
  await nextTick()
  categoryRefs.value[category.id]?.focus?.()
}

function createId() {
  return Date.now() + Math.floor(Math.random() * 1000)
}

function createCategory(title) {
  const category = { id: createId(), title, items: [] }
  categories.value.push(category)
  return category
}

function findEmptyItem() {
  for (const category of categories.value) {
    const item = category.items.find(item => !item.title || !item.title.trim())
    if (item) return { category, item }
  }
  return null
}

async function focusItem(itemId) {
  await nextTick()
  const itemRef = listRefs.value[itemId]
  const el = itemRef?.$el
  itemRef?.focus?.()
  if (el && scroll.value) {
    scroll.value.setScrollPosition('vertical', el.offsetTop, 300)
  }
}

function changeItem(item, type) {
  if (type === 'add') item.qtd++
  if (type === 'sub' && item.qtd > 1) item.qtd--
}

function deleteItem(item, categoryIndex, itemIndex) {
  error.value = item.id
  q.dialog({
    title: 'Atenção',
    message: 'Tem certeza que deseja excluir o item <b>' + item.title + '</b>?',
    html: true,
    ok: { color: 'negative', label: 'Excluir' },
    cancel: { outline: true, label: 'Cancelar', color: 'grey-8' }
  }).onOk(() => {
    categories.value[categoryIndex].items.splice(itemIndex, 1)
    error.value = null
  }).onDismiss(() => { error.value = null })
}

function deleteCategory(category) {
  q.dialog({
    title: 'Atenção',
    message: `Tem certeza que deseja excluir a categoria <b>${category.title}</b> e todos os itens dela?`,
    html: true,
    ok: { color: 'negative', label: 'Excluir' },
    cancel: { outline: true, label: 'Cancelar', color: 'grey-8' }
  }).onOk(() => {
    const index = categories.value.findIndex(item => item.id === category.id)
    if (index !== -1) categories.value.splice(index, 1)
  })
}

function archiveItem(item) {
  const location = findItemLocation(item.id)
  if (!location) return
  archiveList.value.push({
    ...item,
    categoryTitle: location.category.title,
    archivedAt: new Date().toISOString()
  })
  location.category.items.splice(location.itemIndex, 1)
}

function unarchiveItem(item) {
  const index = archiveList.value.findIndex(i => i.id === item.id && i.archivedAt === item.archivedAt)
  if (index === -1) return
  const { archivedAt, categoryTitle, ...original } = archiveList.value[index]
  const targetCategory = categories.value.find(category => category.title === categoryTitle) || categories.value[0] || createCategory(categoryTitle || 'Mercado')
  targetCategory.items.push(original)
  archiveList.value.splice(index, 1)
}

function deleteArchivedItem(item) {
  const index = archiveList.value.findIndex(i => i.id === item.id && i.archivedAt === item.archivedAt)
  if (index !== -1) archiveList.value.splice(index, 1)
}

function findItemLocation(itemId) {
  for (const [categoryIndex, category] of categories.value.entries()) {
    const itemIndex = category.items.findIndex(item => item.id === itemId)
    if (itemIndex !== -1) return { category, categoryIndex, itemIndex }
  }
  return null
}

function startCategoryDrag(event, categoryId) {
  startDrag(event, { type: 'category', categoryId })
}

function startItemDrag(event, categoryId, itemId) {
  startDrag(event, { type: 'item', categoryId, itemId })
}

function startDrag(event, state) {
  event.preventDefault()
  dragState.value = state
  window.addEventListener('pointermove', onPointerMove, { passive: false })
  window.addEventListener('pointerup', stopDrag)
  window.addEventListener('pointercancel', stopDrag)
}

function onPointerMove(event) {
  if (!dragState.value) return
  event.preventDefault()
  const target = document.elementFromPoint(event.clientX, event.clientY)
  if (!target) return

  if (dragState.value.type === 'category') {
    const categoryElement = target.closest('[data-category-id]')
    const targetCategoryId = Number(categoryElement?.dataset.categoryId)
    if (targetCategoryId) moveCategory(dragState.value.categoryId, targetCategoryId)
    return
  }

  const itemElement = target.closest('[data-item-id]')
  if (itemElement) {
    moveItem(
      dragState.value.itemId,
      Number(itemElement.dataset.categoryId),
      Number(itemElement.dataset.itemId)
    )
    return
  }

  const categoryElement = target.closest('[data-category-id]')
  const targetCategoryId = Number(categoryElement?.dataset.categoryId)
  if (targetCategoryId) moveItemToCategoryEnd(dragState.value.itemId, targetCategoryId)
}

function stopDrag() {
  dragState.value = null
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', stopDrag)
  window.removeEventListener('pointercancel', stopDrag)
}

function moveCategory(sourceId, targetId) {
  if (sourceId === targetId) return
  const sourceIndex = categories.value.findIndex(category => category.id === sourceId)
  const targetIndex = categories.value.findIndex(category => category.id === targetId)
  if (sourceIndex === -1 || targetIndex === -1) return
  const [category] = categories.value.splice(sourceIndex, 1)
  categories.value.splice(targetIndex, 0, category)
}

function moveItem(itemId, targetCategoryId, targetItemId) {
  if (itemId === targetItemId) return
  const location = findItemLocation(itemId)
  const targetCategory = categories.value.find(category => category.id === targetCategoryId)
  if (!location || !targetCategory) return
  const [item] = location.category.items.splice(location.itemIndex, 1)
  const targetIndex = targetCategory.items.findIndex(item => item.id === targetItemId)
  targetCategory.items.splice(targetIndex === -1 ? targetCategory.items.length : targetIndex, 0, item)
}

function moveItemToCategoryEnd(itemId, targetCategoryId) {
  const location = findItemLocation(itemId)
  const targetCategory = categories.value.find(category => category.id === targetCategoryId)
  if (!location || !targetCategory) return
  if (location.category.id === targetCategory.id && location.itemIndex === targetCategory.items.length - 1) return
  const [item] = location.category.items.splice(location.itemIndex, 1)
  targetCategory.items.push(item)
}

onBeforeUnmount(stopDrag)
</script>

<style>
.q-field--outlined .q-field__control {
  padding: 0 0 0 10px !important;
}

.category-card {
  border: 1px solid rgba(0, 0, 0, 0.08);
}

.category-header .q-field__control {
  min-height: 32px;
}

.category-title input {
  cursor: text;
}

.drag-handle {
  cursor: grab;
  touch-action: none;
  user-select: none;
}

.drag-handle:active {
  cursor: grabbing;
}

.dragging-card,
.dragging-item {
  opacity: 0.55;
}

.item-row:last-child {
  margin-bottom: 0;
}

.empty-category {
  border: 1px dashed rgba(0, 0, 0, 0.18);
  border-radius: 4px;
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
