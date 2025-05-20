<template>
  <div class="q-pa-md">
    <q-card dark class="q-mb-md">
      <q-card-section>
        <div class="text-h6 q-mb-md">Add New Task</div>
        <div class="row q-col-gutter-md">
          <div class="col">
            <q-input
              v-model="newTask"
              outlined
              dense
              dark
              color="primary"
              placeholder="Enter a new task"
              @keyup.enter="addTask"
            />
          </div>
          <div class="col-auto">
            <q-btn
              color="primary"
              icon="add"
              label="Add"
              @click="addTask"
              :loading="isAddingTask"
            />
          </div>
        </div>
      </q-card-section>
    </q-card>

    <!-- Loading State -->
    <div v-if="isLoading" class="text-center q-pa-md">
      <q-spinner color="primary" size="3em" />
      <div class="text-white q-mt-md">Loading tasks...</div>
    </div>

    <!-- Error State -->
    <q-banner v-if="error" class="bg-negative text-white q-mb-md">
      {{ error }}
      <template v-slot:action>
        <q-btn flat color="white" label="Retry" @click="fetchTasks" />
      </template>
    </q-banner>

    <q-list v-if="!isLoading" bordered separator dark class="rounded-borders">
      <q-item v-for="todo in todos" :key="todo.id" dark>
        <q-item-section avatar>
          <q-checkbox v-model="todo.completed" @update:model-value="updateTask(todo)" color="primary" />
        </q-item-section>
        
        <q-item-section>
          <q-item-label v-if="!todo.editing" :class="{'text-strike': todo.completed}">
            {{ todo.title }}
          </q-item-label>
          <q-input
            v-else
            v-model="todo.title"
            dense
            outlined
            dark
            color="primary"
            autofocus
            @keyup.enter="saveTask(todo)"
            @blur="saveTask(todo)"
          />
        </q-item-section>
        
        <q-item-section side>
          <div class="row items-center">
            <q-btn
              v-if="!todo.editing"
              flat
              round
              color="primary"
              icon="edit"
              @click="startEditing(todo)"
            />
            <q-btn
              flat
              round
              color="negative"
              icon="delete"
              @click="deleteTask(todo)"
              :loading="todo.isDeleting"
            />
          </div>
        </q-item-section>
      </q-item>

      <q-item v-if="todos.length === 0 && !isLoading" dark>
        <q-item-section class="text-center q-py-xl">
          <div class="text-h4 text-white q-mb-md">Get ready to conquer the day</div>
        </q-item-section>
      </q-item>
    </q-list>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const todos = ref([])
const newTask = ref('')
const isLoading = ref(false)
const isAddingTask = ref(false)
const error = ref(null)

const fetchTasks = async () => {
  isLoading.value = true
  error.value = null
  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/todos')
    todos.value = response.data.slice(0, 3).map(todo => ({
      ...todo,
      editing: false,
      isDeleting: false
    }))
  } catch (err) {
    error.value = 'Failed to load tasks. Please try again.'
    console.error('Error fetching tasks:', err)
  } finally {
    isLoading.value = false
  }
}

const addTask = async () => {
  if (!newTask.value.trim()) return

  isAddingTask.value = true
  try {
    const response = await axios.post('https://jsonplaceholder.typicode.com/todos', {
      title: newTask.value,
      completed: false,
      userId: 1
    })
    
    todos.value.unshift({
      ...response.data,
      editing: false,
      isDeleting: false
    })
    newTask.value = ''
  } catch (error) {
    console.error('Failed to add task:', error)
  } finally {
    isAddingTask.value = false
  }
}

// UPDATE - Update existing task
const updateTask = async (todo) => {
  try {
    await axios.put(`https://jsonplaceholder.typicode.com/todos/${todo.id}`, {
      ...todo,
      completed: todo.completed
    })
  } catch (error) {
    console.error('Failed to update task:', error)
    todo.completed = !todo.completed
  }
}

const deleteTask = async (todo) => {
  todo.isDeleting = true
  try {
    await axios.delete(`https://jsonplaceholder.typicode.com/todos/${todo.id}`)
    todos.value = todos.value.filter(t => t.id !== todo.id)
  } catch (error) {
    console.error('Failed to delete task:', error)
  } finally {
    todo.isDeleting = false
  }
}

const startEditing = (todo) => {
  todo.editing = true
}

const saveTask = async (todo) => {
  todo.editing = false
  await updateTask(todo)
}

onMounted(fetchTasks)
</script>

<style lang="scss">
// Styles can be added here if needed
</style>
