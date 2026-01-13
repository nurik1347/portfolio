<script setup>
import { ref } from 'vue';

const emit = defineEmits(['add-task'])

const newTask = ref('')
const priority = ref('medium')

function addTask() {
    if (newTask.value.trim() === '') return

    emit('add-task', {
        text: newTask.value.trim(),
        priority: priority.value
    })
    newTask.value = ''
    priority.value = 'medium'
}
</script>

<template>
    <div class="task-input">
        <input v-model="newTask" type="text" placeholder="Yangi vazifa kiritish..." @keyup.enter="addTask" />

        <select v-model="priority" class="priority-select">
            <option value="low">Pas (Low)</option>
            <option value="medium">O'rta (Medium)</option>
            <option value="high">Yuqori (High)</option>
        </select>
        <button @click="addTask">Qoshish</button>
    </div>
</template>

<style scoped>
.task-input {
  display: flex;
  gap: 8px;
  margin-bottom: 20px;
}

input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.priority-select {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
}

button {
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #45a049;
}
</style>