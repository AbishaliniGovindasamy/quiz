<script setup>
import { ref } from 'vue';
const emit = defineEmits(['start-quiz']);
const props = defineProps(['errorMessage']);

const topic = ref('');
</script>

<template>
  <section class="start-screen container">
    <h2>Choose a topic in ComputerScience  to start testing your knowledge</h2>
    <input v-model="topic" type="text" placeholder="Enter your topic..." />
    <button @click="emit('start-quiz', topic)" :disabled="!topic">
      Start Quiz
    </button>
    <small class="error" v-if="errorMessage">
There was an Error, Please try again Later.
    </small>
  </section>
</template>


<!-- AIzaSyBRNYiVsnjnaTgAadpDPvZWCdE3xokSQXE -->