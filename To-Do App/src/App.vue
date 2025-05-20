<script setup>
import { onMounted } from 'vue'
import { useQuasar } from 'quasar'
import TodoList from './components/TodoList.vue'

const $q = useQuasar()

onMounted(() => {
  $q.dark.set(true)
})
</script>

<template>
  <q-layout view="hHh lpR fFf" class="bg-dark">
    <q-page-container>
      <q-page padding>
        <div class="text-h3 text-center q-my-lg text-primary">To-Do Application</div>
        <TodoList />
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<style lang="scss">
// Add any global styles here
body {
  background-color: #121212;
}
</style>
