<script setup>
import TaskInput from './components/TaskInput.vue'
import { ref, watch, computed, onMounted, watchEffect } from 'vue'

const tasks = ref([])

onMounted(() => {
  const saved = localStorage.getItem('tasks')
  if (saved) tasks.value = JSON.parse(saved)
})

watch(tasks, newVal => {
  localStorage.setItem('tasks', JSON.stringify(newVal))
}, { deep: true })

function addNewTask(payload) {
  let text, priority = 'medium'

  if (typeof payload === 'string') {
    text = payload
  } else if (payload && payload.text) {
    text = payload.text
    priority = payload.priority || 'medium'
  }

  if (!text?.trim()) return

  tasks.value.push({
    id: Date.now(),
    text: text.trim(),
    done: false,
    priority
  })
}

function toggleDone(task) {
  task.done = !task.done
}

function removeTask(taskId) {
  tasks.value = tasks.value.filter(t => t.id !== taskId)
}

const filter = ref('all')

const filteredTasks = computed(() => {
  if (filter.value === 'active') return tasks.value.filter(t => !t.done)
  if (filter.value === 'completed') return tasks.value.filter(t => t.done)
  return tasks.value
})

const remaining = computed(() => tasks.value.filter(t => !t.done).length)

const editingTaskId = ref(null)
const editText = ref('')

function startEditing(task) {
  editingTaskId.value = task.id
  editText.value = task.text
}

function saveEdit(task) {
  if (editText.value.trim() === '') {
    removeTask(task.id)
  } else {
    task.text = editText.value.trim()
  }
  editingTaskId.value = null
}

function cancelEdit() {
  editingTaskId.value = null
}

const theme = ref(localStorage.getItem('theme') || 'light')

watchEffect(() => {
  document.documentElement.classList.remove('light', 'dark')
  document.documentElement.classList.add(theme.value)
  localStorage.setItem('theme', theme.value)
})

function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

function dragStart(event, task) {
  event.dataTransfer.setData('text/plain', task.id)
  event.target.classList.add('dragging')
}

function dragOver(event) {
  event.preventDefault()
}

function drop(event, targetTask) {
  event.preventDefault()
  
  const draggedId = Number(event.dataTransfer.getData('text/plain'))
  const draggedTask = tasks.value.find(t => t.id === draggedId)
  
  if (!draggedTask || draggedTask.id === targetTask.id) return

  const draggedIndex = tasks.value.indexOf(draggedTask)
  tasks.value.splice(draggedIndex, 1)

  const targetIndex = tasks.value.indexOf(targetTask)
  tasks.value.splice(targetIndex, 0, draggedTask)

  document.querySelectorAll('.dragging').forEach(el => el.classList.remove('dragging'))
}

function dragEnd(event) {
  event.target.classList.remove('dragging')
}
</script>

<template>
  <div class="container">
    <button class="theme-toggle" @click="toggleTheme">
      {{ theme === 'light' ? '🌙' : '☀️' }}
    </button>

    <h1>Mening vazifalarim</h1>

    <TaskInput @add-task="addNewTask" />

    <div v-if="tasks.length === 0" class="empty-state">
      Hozircha vazifalar yo'q... Yangisini qo'shing! 😄
    </div>

    <ul class="task-list" v-else>
      <li 
        v-for="task in filteredTasks" 
        :key="task.id"
        :class="{
          done: task.done,
          editing: editingTaskId === task.id,
          'priority-high': task.priority === 'high' && !task.done,
          'priority-medium': task.priority === 'medium' && !task.done,
          'priority-low': task.priority === 'low' && !task.done
        }"
        draggable="true"
        @dragstart="dragStart($event, task)"
        @dragover="dragOver"
        @drop="drop($event, task)"
        @dragend="dragEnd"
        @dblclick="startEditing(task)"
      >
        <input type="checkbox" :checked="task.done" @change="toggleDone(task)" />

        <div class="task-content">
          <input 
            v-if="editingTaskId === task.id" 
            v-model="editText" 
            type="text" 
            class="edit-input"
            @blur="saveEdit(task)" 
            @keyup.enter="saveEdit(task)" 
            @keyup.escape="cancelEdit" 
            autofocus 
          />
          <span v-else class="task-text">
            <span class="priority-indicator"></span>
            {{ task.text }}
          </span>
        </div>

        <button class="delete-btn" @click="removeTask(task.id)">🗑️</button>
      </li>
    </ul>

    <div class="filters" v-if="tasks.length > 0">
      <button :class="{ active: filter === 'all' }" @click="filter = 'all'">Hammasi</button>
      <button :class="{ active: filter === 'active' }" @click="filter = 'active'">Faol ({{ remaining }})</button>
      <button :class="{ active: filter === 'completed' }" @click="filter = 'completed'">Bajarilgan</button>
    </div>
  </div>
</template>

<style>
:root {
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --text-primary: #2c3e50;
  --text-secondary: #555555;
  --border: #dddddd;
  --input-bg: #ffffff;
  --shadow: rgba(0, 0, 0, 0.10);
  --empty-text: #999999;
  --hover-bg: #f0f4f8;
  --checkbox-bg: #ffffff;
  --checkbox-border: #cccccc;
  --accent: #4caf50;
}

.dark {
  --bg-primary: #1a1a1a;
  --bg-secondary: #252525;
  --text-primary: #e0e0e0;
  --text-secondary: #aaaaaa;
  --border: #444444;
  --input-bg: #2d2d2d;
  --shadow: rgba(0, 0, 0, 0.50);
  --empty-text: #777777;
  --hover-bg: #333333;
  --checkbox-bg: #333333;
  --checkbox-border: #555555;
  --accent: #66bb6a;
}

.container {
  max-width: 600px;
  margin: 40px auto;
  padding: 24px;
  background: var(--bg-primary);
  color: var(--text-primary);
  border-radius: 12px;
  box-shadow: 0 4px 20px var(--shadow);
  transition: all 0.35s ease;
  position: relative;
}

h1 {
  text-align: center;
  color: var(--text-primary);
  margin: 0 0 32px;
  font-weight: 600;
  font-size: 2rem;
}

.theme-toggle {
  position: absolute;
  top: 20px;
  right: 20px;
  background: none;
  border: none;
  font-size: 1.6rem;
  cursor: pointer;
  padding: 8px;
  border-radius: 50%;
  color: var(--text-primary);
  transition: background 0.3s, transform 0.2s;
}

.theme-toggle:hover {
  background: rgba(128, 128, 128, 0.15);
  transform: scale(1.15);
}

.task-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.task-list li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  background: var(--bg-secondary);
  margin: 8px 0;
  border-radius: 10px;
  transition: all 0.3s ease;
  min-height: 52px;
  color: var(--text-primary);
  user-select: none;
}

.task-list li:hover {
  background: var(--hover-bg);
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}

.task-list li.done {
  opacity: 0.85;
}

.task-list li.done .task-text {
  text-decoration: line-through;
  color: var(--text-secondary);
}

.task-list input[type="checkbox"] {
  margin-right: 14px;
  width: 20px;
  height: 20px;
  flex-shrink: 0;
  background: var(--checkbox-bg);
  border: 2px solid var(--checkbox-border);
  border-radius: 4px;
  accent-color: var(--accent);
  cursor: pointer;
  transition: all 0.2s;
}

.task-content {
  flex: 1 1 auto;
  min-width: 0;
  overflow: hidden;
}

.task-text {
  display: block;
  white-space: pre-wrap;
  word-break: break-word;
  line-height: 1.45;
  color: var(--text-primary);
  font-size: 1.05rem;
}

.edit-input {
  width: 100%;
  padding: 8px 12px;
  font-size: 1.05rem;
  border: 2px solid var(--accent);
  border-radius: 8px;
  outline: none;
  background: var(--input-bg);
  color: var(--text-primary);
  box-sizing: border-box;
  transition: border-color 0.2s;
}

.edit-input:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.15);
}

.task-list li.editing {
  background: #fff3e0;
  box-shadow: 0 0 0 2px #ffd180 inset;
}

.task-list li:hover .task-text {
  cursor: text;
  position: relative;
}

.task-list li:hover .task-text::after {
  content: " ✏️";
  font-size: 0.9rem;
  margin-left: 8px;
  opacity: 0.5;
  vertical-align: middle;
}

.delete-btn {
  background: #ff5252;
  color: white;
  border: none;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  cursor: pointer;
  font-size: 1.1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  flex-shrink: 0;
  margin-left: 12px;
}

.delete-btn:hover {
  background: #e53935;
  transform: scale(1.1);
}

.priority-indicator {
  display: inline-block;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 10px;
  flex-shrink: 0;
}

.priority-high .priority-indicator { background: #ff6b6b; }
.priority-medium .priority-indicator { background: #ffb74d; }
.priority-low .priority-indicator { background: #66bb6a; }

.dark .priority-high .priority-indicator { background: #ff5252; }
.dark .priority-medium .priority-indicator { background: #ffb74d; }
.dark .priority-low .priority-indicator { background: #66bb6a; }

.task-list li.done .priority-indicator {
  opacity: 0.45;
}

.filters {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-top: 28px;
  flex-wrap: wrap;
}

.filters button {
  padding: 8px 18px;
  border: 1px solid var(--border);
  border-radius: 8px;
  background: var(--input-bg);
  color: var(--text-primary);
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 500;
  transition: all 0.25s;
}

.filters button:hover {
  background: var(--hover-bg);
  transform: translateY(-1px);
}

.filters button.active {
  background: var(--accent);
  color: white;
  border-color: var(--accent);
  box-shadow: 0 2px 8px rgba(76, 175, 80, 0.3);
}

.empty-state {
  text-align: center;
  color: var(--empty-text);
  padding: 60px 0;
  font-size: 1.2rem;
  font-style: italic;
}

.task-list li[draggable="true"] {
  cursor: grab;
}

.task-list li.dragging {
  opacity: 0.45;
  transform: scale(0.98);
  background: var(--hover-bg);
  box-shadow: 0 6px 16px rgba(0,0,0,0.2);
}
</style>