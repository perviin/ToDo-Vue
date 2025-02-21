<script setup>
import { ref, onMounted, computed, watch } from "vue";

const todos = ref([]);
const name = ref("");
const input_content = ref("");

// Trier les tâches du plus récent au plus ancien
const todos_asc = computed(() => [...todos.value].sort((a, b) => b.createdAt - a.createdAt));

const addTodo = () => {
  if (input_content.value.trim() === "") return;

  todos.value = [
    ...todos.value,
    {
      content: input_content.value,
      done: false,
      createdAt: Date.now(),
    },
  ];

  input_content.value = "";
};

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo);
};

// Sauvegarde les tâches et le nom dans le localStorage
watch(
  todos,
  (newVal) => {
    console.log("Mise à jour du localStorage :", newVal);
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  { deep: true }
);

watch(name, (newVal) => {
  localStorage.setItem("name", newVal);
});

// Récupération des valeurs au chargement
onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  try {
    todos.value = JSON.parse(localStorage.getItem("todos")) || [];
  } catch (e) {
    todos.value = [];
  }
});

const onEnter = (event) => {
 if (event.key === "Enter") {
  addTodo();
 } 
};

const progress = computed(() => {
  if (todos.value.length === 0) return 0;
  const completedTasks = todos.value.filter(todo => todo.done).length;
  return Math.round((completedTasks / todos.value.length) * 100);
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">Bonjour, <input type="text" placeholder="Votre nom" v-model="name" /></h2>
    </section>

    <section>
      <form @submit.prevent="addTodo">
        <h4>Quelles sont vos tâches à effectuer ?</h4>
        <input type="text" placeholder="Insérez vos tâches du jour" v-model="input_content" />
        <input type="submit" 
         value="Ajouter une tâche" 
         @keydown.enter="onEnter"
        />
      </form>
    </section>

    <section class="progress-container">
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: progress + '%' }"></div>
      </div>
      <p>{{ progress }}% complété</p>
    </section>

    <section class="todo-list">
      <h3>LISTE DES TÂCHES</h3>
      <div class="list">
        <div v-for="todo in todos_asc" :key="todo.createdAt" :class="{ 'todo-item': true, 'done': todo.done }">
          <label>
            <input type="checkbox" v-model="todo.done" />
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
