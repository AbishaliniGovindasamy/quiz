<script setup>
import StartScreen from './components/StartScreen.vue';
import Loader from './components/Loader.vue';
import Quiz from './components/Quiz.vue';
import Result from './components/Result.vue';
import { GoogleGenerativeAI, SchemaType } from "@google/generative-ai";
import { ref } from 'vue';


const questions = ref([]);
const status = ref("start");
const isLoading = ref(false);
const errorMessage = ref('');

const userAnswers = ref([]);
const restartQuiz = ()=>{

  questions.value='';
  status.value='start';
  userAnswers.value=[];
  errorMessage.value='';
}

const storeAnswer = (answer) =>{
  userAnswers.value.push(answer);
}


const shuffleOptions = (correct, incorrect) => {
  const safeIncorrect = Array.isArray(incorrect) ? incorrect : [];
  const options = [correct, ...safeIncorrect];
  return options.sort(() => Math.random() - 0.5);
};

const startQuiz = async (topic) => {
  status.value = 'loading';
  if (isLoading.value) return;

  isLoading.value = true;
  questions.value = [];

    const genAi = new GoogleGenerativeAI("AIzaSyBRNYiVsnjnaTgAadpDPvZWCdE3xokSQXE");

    const schema = {
      description: "List of multiple-choice quiz questions",
      type: SchemaType.ARRAY,
      items: {
        type: SchemaType.OBJECT,
        properties: {
          type: { type: SchemaType.STRING, nullable: false },
          difficulty: { type: SchemaType.STRING, nullable: false },
          category: { type: SchemaType.STRING, nullable: false },
          question: { type: SchemaType.STRING, nullable: false },
          correct_answer: { type: SchemaType.STRING, nullable: false },
          incorrect_answers: {
            type: SchemaType.ARRAY,
            items: { type: SchemaType.STRING },
            nullable: false,
          },
        },
        required: [
          "type",
          "difficulty",
          "category",
          "question",
          "correct_answer",
          "incorrect_answers",
        ],
      },
    };

    const model = genAi.getGenerativeModel({
      model: "gemini-1.5-flash",
      generationConfig: {
        responseMimeType: "application/json",
        responseSchema: schema,
      },
    });
  try {

    const result = await model.generateContent(
      `Create 5 multiple-choice quiz questions about ${topic}. Difficulty: Easy to Medium.`
    );

    const text = result.response.text();
    const json = JSON.parse(text);

    questions.value = json;
    status.value = 'ready';
  } catch (error) {
    console.error("Error:", error);
    questions.value = [
      { question: error.message || "Unexpected error occurred." }
    ];
    errorMessage.value = error;
    status.value = 'start';
  } finally {
    isLoading.value = false;
  }
};
</script>

<template>
  <div id="app">
    <header>
      <div class="container">
        <img src="./assets/logo.png" class="logo" />
        <h1>Quiz Generator</h1>
      </div>
    </header>

    <StartScreen v-if="status === 'start'" :errorMessage="errorMessage" @start-quiz="startQuiz" :isLoading="isLoading" />
    <Loader v-if="status === 'loading'" />
    <Quiz v-if="status === 'ready'" @end-quiz="status='finished'"  @store-answer="storeAnswer"  :questions="questions" />
    <Result v-if="status == 'finished'" @restart-quiz="restartQuiz" :userAnswers="userAnswers" />
 </div>
</template>
