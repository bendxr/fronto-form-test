<template>
  <div class="app-container">
    <header>
      <h1>Vue.js Inter-dependent Form Fields</h1>
      <p>A demonstration of form fields that dynamically interact with each other</p>
    </header>
    <DependentForm />
  </div>
</template>

<script setup>
import DependentForm from './components/DependentForm.vue'
</script>

<style scoped>
.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

header {
  text-align: center;
  margin-bottom: 2rem;
}

h1 {
  color: #42b983;
  margin-bottom: 0.5rem;
}

p {
  color: #666;
  font-size: 1.1rem;
}
</style>
