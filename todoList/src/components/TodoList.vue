<script lang="ts" setup>
import ListItem from './ListItem.vue'
import { ref, onMounted, computed } from 'vue'

const apiUrl = 'http://localhost:3000/tasks'

interface Item {
  id: number
  title: string
  checked: boolean
}

const items = ref<Item[]>([])
const newTaskTitle = ref('')
const editingId = ref<number | null>(null)

const fetchTasks = async () => {
  const res = await fetch(apiUrl)
  items.value = await res.json()
}

const addTask = async () => {
  if (!newTaskTitle.value.trim()) return
  const res = await fetch(apiUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: newTaskTitle.value, checked: false }),
  })
  const newTask = await res.json()
  items.value.push(newTask)
  newTaskTitle.value = ''
}

const editTask = (id: number) => {
  const task = items.value.find((item) => item.id === id)
  if (!task) return
  newTaskTitle.value = task.title
  editingId.value = id
}

const saveEdit = async () => {
  if (editingId.value === null || !newTaskTitle.value.trim()) return
  const res = await fetch(`${apiUrl}/${editingId.value}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: newTaskTitle.value }),
  })
  if (res.ok) {
    const idx = items.value.findIndex((item) => item.id === editingId.value)
    if (idx !== -1) items.value[idx].title = newTaskTitle.value
    newTaskTitle.value = ''
    editingId.value = null
  }
}

const deleteTask = async (id: number) => {
  await fetch(`${apiUrl}/${id}`, { method: 'DELETE' })
  items.value = items.value.filter((item) => item.id !== id)
}

const toggleChecked = async (item: Item) => {
  const updated = { ...item, checked: !item.checked }
  await fetch(`${apiUrl}/${item.id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ checked: updated.checked }),
  })
  item.checked = updated.checked
}

const sortedList = computed(() =>
  [...items.value].sort((a, b) => (a.checked ? 1 : 0) - (b.checked ? 1 : 0)),
)

onMounted(fetchTasks)
</script>

<template>
  <div class="main container py-4">
    <div class="row justify-content-center">
      <div class="col-md-8">
        <form class="input-group mb-4" @submit.prevent="editingId ? saveEdit() : addTask()">
          <input v-model="newTaskTitle" class="form-control" placeholder="Nueva tarea" />
          <button class="btn btn-primary" type="submit">
            {{ editingId ? 'Guardar' : 'Agregar' }}
          </button>
          <button
            v-if="editingId"
            class="btn btn-secondary ms-2"
            type="button"
            @click="
              () => {
                editingId = null
                newTaskTitle = ''
              }
            "
          >
            Cancelar
          </button>
        </form>
        <ul class="list-group">
          <li
            v-for="item in sortedList"
            :key="item.id"
            class="list-group-item d-flex align-items-center justify-content-between"
          >
            <div class="d-flex align-items-center">
              <ListItem :is-checked="item.checked" @click.prevent="toggleChecked(item)">
                {{ item.title }}
              </ListItem>
            </div>
            <button class="btn btn-primary btn-sm" @click="editTask(item.id)">Editar</button>
            <button class="btn btn-danger btn-sm" @click="deleteTask(item.id)">Eliminar</button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
.list-group-item {
  transition: background 0.2s;
}
.list-group-item:hover {
  background: #f8f9fa;
}
.input-group input {
  border-radius: 0.375rem 0 0 0.375rem;
}
.input-group .btn {
  border-radius: 0 0.375rem 0.375rem 0;
}
</style>
