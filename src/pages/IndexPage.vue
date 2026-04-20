<template>
  <q-page class="full-width full-height">
    <q-scroll-area class="full-width " style="height:calc(100% - 60px)" ref="scroll">
      <div style="height:50px;margin:5px 10px" v-for="(item, i) in list" :key="item.id">
        <q-input :ref="el => listRefs[i] = el" :for="item.id.toString()" v-model="item.title" outlined dense :rules="[
          val => !!val || 'Não deixe a descrição vazia'
        ]" :bg-color="`${error === item.id ? 'red-4' : 'transparent'}`"
          :input-style="{ padding: '10px', textDecoration: item.added ? 'line-through' : '' }">
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
                <q-item-section class="text-uppercase font-xs text-info">
                  Arquivar Item
                </q-item-section>
              </q-item>
            </q-list>
          </q-menu>
          <template #append>
            <div class="flex flex-center" style="width:80px;">
              <q-btn @click="changeItem(item, 'sub')" icon="keyboard_arrow_left" dense color="primary" flat size="sm" />
              <input @keyup.enter="addItem" v-model="item.qtd" mask="#" inputmode="numeric" class="inputQtd" />
              <q-btn @click="changeItem(item, 'add')" icon="keyboard_arrow_right" dense color="primary" size="sm"
                flat />
            </div>
          </template>
          <template #prepend>
            <q-checkbox icon="arrow_upward" color="primary" unelevated dense size="lg" v-model="item.added" />
          </template>
        </q-input>
      </div>
    </q-scroll-area>
    <div class="full-width q-px-sm">
      <q-btn @click="addItem" icon="add" label="Adicionar item" outline color="primary" class="full-width" />
    </div>
    <!-- <q-form style="height:50px;margin:5px 10px" @submit.prevent="addItem" class="flex flex-start">
      <q-input style="flex:1" v-model="currentItem.title" outlined dense
        :rules="[val => !!val || 'Por favor digite a descrição do item']">
        <template #append>
          <div class="flex flex-center" style="width:80px;">
            <q-btn @click="changeItem(currentItem, 'sub')" icon="keyboard_arrow_left" dense color="primary" flat
              size="sm" />
            <input v-model="currentItem.qtd" mask="#" inputmode="numeric" class="inputQtd" />
            <q-btn @click="changeItem(currentItem, 'add')" icon="keyboard_arrow_right" dense color="primary" size="sm"
              flat />

          </div>
        </template>
      </q-input>
      <q-btn icon="arrow_upward" color="primary" unelevated dense type="submit"
        style="height:40px;width:40px;margin-left:5px" />
    </q-form> -->
  </q-page>
</template>

<script setup>
import { useStorage } from '@vueuse/core';
import { useQuasar } from 'quasar';
import { nextTick, ref } from 'vue';

const openAdd = ref(false)
const scroll = ref(null)
const listRefs = ref([])
const q = useQuasar()
const currentItem = ref({ id: null, title: null, qtd: 1, added: false })
const list = useStorage('list', [
  { id: 1, title: 'Açucar 5kg', qtd: 3, added: false }
])
const archiveList = useStorage('list', [
  { id: 1, title: 'Açucar 5kg', qtd: 3, added: false, date: null }
])


async function addItem() {
  // 🔥 1. verificar item vazio
  const indexEmpty = list.value.findIndex(item => !item.title || !item.title.trim())

  if (indexEmpty !== -1) {
    const item = list.value[indexEmpty]


    await nextTick()

    // 🔥 scroll até o item com erro
    const el = listRefs.value[indexEmpty]?.$el
    listRefs.value[indexEmpty]?.focus()
    if (el && scroll.value) {
      scroll.value.setScrollPosition(
        'vertical',
        el.offsetTop,
        300
      )
    }

    // 🔥 focar no input com erro
    listRefs.value[indexEmpty]?.focus?.()

    return // 🚫 não adiciona novo item
  }

  // =========================
  // ✅ fluxo normal
  // =========================

  currentItem.value.id = list.value.length + 1

  list.value.push({ ...currentItem.value })

  await nextTick()

  const el = listRefs.value[listRefs.value.length - 1]?.$el

  if (el && scroll.value) {
    scroll.value.setScrollPosition(
      'vertical',
      el.offsetTop,
      300
    )
  }

  listRefs.value[listRefs.value.length - 1]?.focus?.()

  currentItem.value = { id: null, title: null, qtd: 1, added: false }
}
function changeItem(item, type) {
  if (type == 'add') item.qtd++;
  if (type == 'sub' && item.qtd > 1) {
    item.qtd--
  }
}
const error = ref(null)
function deleteItem(item, index) {
  error.value = item.id;
  q.dialog({
    title: 'Atenção',
    message: 'Tem certeza que deseja excluir item <b>' + item.title + '</b> ?',
    html: true,
    ok: {
      color: 'negative',
      label: 'Excluir'
    },
    cancel: {
      outline: true,
      label: 'Cancelar',
      color: 'grei-8'
    }
  }).onOk(() => {
    list.value.splice(index, 1)
    error.value = null
  }).onDismiss(() => error.value = null)
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
