<script setup>
import { computed } from 'vue';

const props = defineProps(['userAnswers']);
const emit = defineEmits(['restart-quiz']);

const correctAnswerCount = computed(() => {
  return props.userAnswers.filter(
    answer => answer.userAnswer === answer.question.correct_answer
  ).length;
});
</script>

<template>
  <section class="result-screen container">
    <h1 v-if="correctAnswerCount === props.userAnswers.length">
      🎉 Congratulations!!
    </h1>
    <h1>
      You have answered {{ correctAnswerCount }} out of {{ props.userAnswers.length }} questions correctly
    </h1>

    <ul>
      <li
        v-for="(answer, index) in props.userAnswers"
        :key="index"
        :class="answer.question.correct_answer === answer.userAnswer ? 'correct' : 'incorrect'"
      >
        <p v-html="answer.question.correct_answer === answer.userAnswer ? '&#9989;' : '&#10060;'"></p>
        <b>{{ answer.question.question }}</b>
        <p>Your answer: <code>{{ answer.userAnswer }}</code></p>
        <p>Correct answer: <code>{{ answer.question.correct_answer }}</code></p>
      </li>
    </ul>

    <button @click="emit('restart-quiz')">Create a New Quiz</button>
  </section>
</template>
