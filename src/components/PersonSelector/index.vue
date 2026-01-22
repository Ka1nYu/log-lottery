<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { usePersonConfig } from '@/store/personConfig'

const props = defineProps<{
  selectedIds: string[]
}>()

const emit = defineEmits<{
  (e: 'submit', ids: string[]): void
  (e: 'close'): void
}>()

const { t } = useI18n()
const personConfig = usePersonConfig()
const dialogRef = ref<HTMLDialogElement>()
const searchQuery = ref('')
const internalSelectedIds = ref<string[]>([])

// Initialize internal state when props change
watch(() => props.selectedIds, (newVal) => {
  internalSelectedIds.value = [...(newVal || [])]
}, { immediate: true })

const filteredPeople = computed(() => {
  const query = searchQuery.value.toLowerCase()
  return personConfig.getAllPersonList.filter(p => 
    p.name.toLowerCase().includes(query) || 
    p.uid.toLowerCase().includes(query) ||
    (p.department && p.department.toLowerCase().includes(query))
  )
})

function showModal() {
  dialogRef.value?.showModal()
}

function close() {
  dialogRef.value?.close()
  emit('close')
}

function save() {
  emit('submit', internalSelectedIds.value)
  close()
}

function toggleSelection(uid: string) {
  const index = internalSelectedIds.value.indexOf(uid)
  if (index === -1) {
    internalSelectedIds.value.push(uid)
  } else {
    internalSelectedIds.value.splice(index, 1)
  }
}

function isSelected(uid: string) {
  return internalSelectedIds.value.includes(uid)
}

defineExpose({ showModal, close })
</script>

<template>
  <dialog ref="dialogRef" class="modal">
    <div class="modal-box w-11/12 max-w-5xl h-[80vh] flex flex-col">
      <h3 class="font-bold text-lg mb-4">{{ t('button.setting') }} {{ t('viewTitle.personManagement') }}</h3>
      
      <!-- Search -->
      <div class="form-control mb-4">
        <input 
          v-model="searchQuery" 
          type="text" 
          :placeholder="t('placeHolder.name')" 
          class="input input-bordered w-full" 
        />
      </div>

      <!-- List -->
      <div class="flex-1 overflow-y-auto">
        <table class="table table-pin-rows">
          <thead>
            <tr>
              <th>
                <label>
                  <input type="checkbox" class="checkbox" disabled />
                </label>
              </th>
              <th>UID</th>
              <th>{{ t('table.name') }}</th>
              <th>{{ t('table.department') }}</th>
              <th>{{ t('table.identity') }}</th>
              <th>{{ t('table.status') }}</th>
            </tr>
          </thead>
          <tbody>
            <tr 
              v-for="person in filteredPeople" 
              :key="person.id" 
              class="hover cursor-pointer" 
              @click="toggleSelection(person.uid)"
            >
              <th>
                <label>
                  <input 
                    type="checkbox" 
                    class="checkbox" 
                    :checked="isSelected(person.uid)" 
                    @click.stop="toggleSelection(person.uid)"
                  />
                </label>
              </th>
              <td>{{ person.uid }}</td>
              <td>{{ person.name }}</td>
              <td>{{ person.department }}</td>
              <td>{{ person.identity }}</td>
              <td>
                <span v-if="person.isWin" class="badge badge-success">{{ t('table.alreadyWin') }}</span>
                <span v-else class="badge badge-ghost">{{ t('table.notWin') }}</span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="modal-action">
        <button class="btn" @click="close">{{ t('button.cancel') }}</button>
        <button class="btn btn-primary" @click="save">{{ t('button.confirm') }}</button>
      </div>
    </div>
    <form method="dialog" class="modal-backdrop">
      <button @click="close">close</button>
    </form>
  </dialog>
</template>
