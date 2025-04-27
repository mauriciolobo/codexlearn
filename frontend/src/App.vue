<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-4">
    <div class="w-full max-w-md bg-white rounded-lg shadow-md p-6">
      <h1 class="text-2xl font-bold text-center mb-4">Fancy Todo App</h1>
      <div class="flex mb-4">
        <input
          v-model="newTodo"
          @keyup.enter="addTodo"
          type="text"
          placeholder="What needs to be done?"
          class="flex-1 px-3 py-2 border border-gray-300 rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <button
          @click="addTodo"
          class="px-4 bg-blue-500 text-white rounded-r hover:bg-blue-600 transition"
        >
          Add
        </button>
      </div>
      <div class="flex justify-center mb-4 space-x-2">
        <button
          :class="filter === 'all' ? activeClass : defaultClass"
          @click="filter = 'all'"
        >
          All
        </button>
        <button
          :class="filter === 'active' ? activeClass : defaultClass"
          @click="filter = 'active'"
        >
          Active
        </button>
        <button
          :class="filter === 'completed' ? activeClass : defaultClass"
          @click="filter = 'completed'"
        >
          Completed
        </button>
      </div>
      <transition-group name="list" tag="ul" class="divide-y divide-gray-200 mb-4">
        <li
          v-for="todo in filteredTodos"
          :key="todo.id"
          class="flex items-center justify-between py-2"
        >
          <div class="flex items-center space-x-2">
            <input
              type="checkbox"
              v-model="todo.completed"
              class="h-4 w-4 text-blue-600 border-gray-300 rounded focus:ring-blue-500"
            />
            <span :class="{ 'line-through text-gray-400': todo.completed }">
              {{ todo.text }}
            </span>
          </div>
          <button
            @click="removeTodo(todo.id)"
            class="text-red-500 hover:text-red-600 transition"
          >
            ×
          </button>
        </li>
      </transition-group>
      <div class="flex justify-between items-center text-sm text-gray-600">
        <span>{{ remaining }} items left</span>
        <button @click="clearCompleted" class="hover:underline">
          Clear completed
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
}

const STORAGE_KEY = 'fancy-todo-app'

const todos = ref<Todo[]>([])
const newTodo = ref('')
const filter = ref<'all' | 'active' | 'completed'>('all')

const filteredTodos = computed(() => {
  if (filter.value === 'active') {
    return todos.value.filter((t) => !t.completed)
  }
  if (filter.value === 'completed') {
    return todos.value.filter((t) => t.completed)
  }
  return todos.value
})

const remaining = computed(() => todos.value.filter((t) => !t.completed).length)

function saveTodos() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
}

function loadTodos() {
  const data = localStorage.getItem(STORAGE_KEY)
  if (data) {
    try {
      todos.value = JSON.parse(data)
    } catch {
      todos.value = []
    }
  }
}

function addTodo() {
  const text = newTodo.value.trim()
  if (!text) return
  todos.value.push({ id: Date.now(), text, completed: false })
  newTodo.value = ''
  saveTodos()
}

function removeTodo(id: number) {
  todos.value = todos.value.filter((t) => t.id !== id)
  saveTodos()
}

function clearCompleted() {
  todos.value = todos.value.filter((t) => !t.completed)
  saveTodos()
}

onMounted(loadTodos)
watch(todos, saveTodos, { deep: true })

const activeClass =
  'px-3 py-1 rounded bg-blue-500 text-white transition'
const defaultClass =
  'px-3 py-1 rounded bg-gray-200 text-gray-700 hover:bg-gray-300 transition'
</script>

<style scoped>
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}
</style>
