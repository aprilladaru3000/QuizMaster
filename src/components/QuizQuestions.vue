<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

// --- Data Kuis ---
// Best practice: Data ini bisa dipindahkan ke file terpisah (misalnya, quizData.js)
const categories = [
  {
    name: 'Science',
    questions: [
      {
        text: 'What planet is known as the Red Planet?',
        options: ['Earth', 'Mars', 'Jupiter', 'Venus'],
        answer: 'Mars',
        hint: 'It is the fourth planet from the Sun.',
        explanation: 'Mars is called the Red Planet because of its reddish appearance due to iron oxide on its surface.'
      },
      {
        text: 'What is the chemical symbol for water?',
        options: ['O2', 'H2O', 'CO2', 'NaCl'],
        answer: 'H2O',
        hint: 'It contains two hydrogen atoms and one oxygen atom.',
        explanation: 'H2O represents the molecular structure of water, with two hydrogen (H) atoms and one oxygen (O) atom.'
      }
    ]
  },
  {
    name: 'History',
    questions: [
      {
        text: 'Who was the first President of the United States?',
        options: ['Abraham Lincoln', 'George Washington', 'John Adams', 'Thomas Jefferson'],
        answer: 'George Washington',
        hint: 'He is featured on the one-dollar bill.',
        explanation: 'George Washington served as the first President of the United States from 1789 to 1797.'
      }
    ]
  }
]

// --- State Management ---
const selectedCategory = ref(categories[0])
const quizState = reactive({
  currentQuestionIndex: 0,
  selectedOption: null,
  showHint: false,
  isAnswered: false,
  score: 0,
  timer: 15,
  quizFinished: false
})
let timerInterval = null

// --- Leaderboard ---
const leaderboard = ref([])

// --- Computed Properties ---
const currentQuestion = computed(() => selectedCategory.value.questions[quizState.currentQuestionIndex])
const isLastQuestion = computed(() => quizState.currentQuestionIndex === selectedCategory.value.questions.length - 1)

// --- Functions ---
function startQuiz(category) {
  selectedCategory.value = category
  quizState.currentQuestionIndex = 0
  quizState.score = 0
  quizState.quizFinished = false
  resetCurrentQuestionState()
  startTimer()
}

function selectOption(option) {
  if (quizState.isAnswered) return

  quizState.isAnswered = true
  quizState.selectedOption = option
  stopTimer()

  if (option === currentQuestion.value.answer) {
    quizState.score++
  }
}

function nextQuestion() {
  if (!isLastQuestion.value) {
    quizState.currentQuestionIndex++
    resetCurrentQuestionState()
    startTimer()
  } else {
    finishQuiz()
  }
}

function finishQuiz() {
  quizState.quizFinished = true
  stopTimer()
  updateLeaderboard()
}

function resetCurrentQuestionState() {
  quizState.selectedOption = null
  quizState.isAnswered = false
  quizState.showHint = false
  quizState.timer = 15
}

function updateLeaderboard() {
  // TODO: Minta nama pengguna untuk leaderboard yang lebih baik
  leaderboard.value.push({
    name: 'User', 
    score: quizState.score,
    category: selectedCategory.value.name
  })
  leaderboard.value.sort((a, b) => b.score - a.score)
}

// --- Timer Logic ---
function startTimer() {
  stopTimer() // Pastikan tidak ada timer ganda
  timerInterval = setInterval(() => {
    if (quizState.timer > 0) {
      quizState.timer--
    } else {
      selectOption(null) // Waktu habis, anggap jawaban salah
    }
  }, 1000)
}

function stopTimer() {
  clearInterval(timerInterval)
}

// --- Lifecycle Hooks ---
onMounted(startTimer)
onUnmounted(stopTimer)

// --- Dynamic Styling ---
function getOptionClass(option) {
  if (!quizState.isAnswered) {
    return 'bg-white hover:bg-gray-100'
  }
  if (option === currentQuestion.value.answer) {
    return 'bg-green-200 border-green-500 text-green-800'
  }
  if (option === quizState.selectedOption) {
    return 'bg-red-200 border-red-500 text-red-800'
  }
  return 'bg-white opacity-60'
}
</script>

<template>
  <div class="font-sans max-w-2xl mx-auto p-4 md:p-6">
    <div class="mb-4">
      <h2 class="text-xl font-bold mb-2 text-gray-700">Choose a Category:</h2>
      <div class="flex flex-wrap gap-2">
        <button
          v-for="cat in categories"
          :key="cat.name"
          @click="startQuiz(cat)"
          :class="[
            'px-4 py-2 rounded-lg font-semibold transition-colors duration-200',
            cat === selectedCategory ? 'bg-blue-600 text-white' : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
          ]">
          {{ cat.name }}
        </button>
      </div>
    </div>
    
    <div v-if="!quizState.quizFinished">
      <div class="flex justify-between items-center bg-gray-100 p-3 rounded-lg mb-4">
        <span class="text-lg font-bold text-gray-800">Score: {{ quizState.score }}</span>
        <span class="text-lg font-bold" :class="quizState.timer < 6 ? 'text-red-500' : 'text-gray-800'">
          Time: {{ quizState.timer }}s
        </span>
      </div>

      <div class="bg-white p-6 rounded-lg shadow-md">
        <p class="text-sm font-semibold text-blue-600">
          Question {{ quizState.currentQuestionIndex + 1 }} of {{ selectedCategory.questions.length }}
        </p>
        <h3 class="text-2xl font-semibold my-3 text-gray-900">{{ currentQuestion.text }}</h3>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-3 my-4">
          <button
            v-for="opt in currentQuestion.options"
            :key="opt"
            @click="selectOption(opt)"
            :disabled="quizState.isAnswered"
            :class="['w-full text-left p-3 border rounded-lg transition-colors duration-200 font-medium', getOptionClass(opt)]">
            {{ opt }}
          </button>
        </div>

        <div class="mt-4">
          <button
            v-if="!quizState.showHint && !quizState.isAnswered"
            @click="quizState.showHint = true"
            class="text-sm text-blue-500 hover:underline">
            Need a hint?
          </button>
          
          <p v-if="quizState.showHint && !quizState.isAnswered" class="text-gray-600 italic">
            Hint: {{ currentQuestion.hint }}
          </p>

          <div v-if="quizState.isAnswered" class="bg-gray-50 p-4 rounded-md border-l-4 border-blue-400">
            <p class="font-bold text-lg mb-2">
              <span v-if="quizState.selectedOption === currentQuestion.answer" class="text-green-600">Correct!</span>
              <span v-else class="text-red-600">Incorrect!</span>
            </p>
            <p class="text-gray-700">{{ currentQuestion.explanation }}</p>
          </div>
        </div>
        
        <div v-if="quizState.isAnswered" class="mt-6 text-right">
            <button @click="nextQuestion" class="px-6 py-2 bg-blue-600 text-white font-bold rounded-lg hover:bg-blue-700">
              {{ isLastQuestion ? 'Finish Quiz' : 'Next Question' }}
            </button>
        </div>
      </div>
    </div>
    
    <div v-else class="text-center bg-white p-8 rounded-lg shadow-md">
      <h2 class="text-3xl font-bold text-blue-600 mb-2">Quiz Finished!</h2>
      <p class="text-xl text-gray-700 mb-4">
        Your final score is: <span class="font-extrabold">{{ quizState.score }}</span>
      </p>
      <button @click="startQuiz(selectedCategory)" class="px-6 py-2 bg-green-500 text-white font-bold rounded-lg hover:bg-green-600">
        Play Again
      </button>
    </div>

    <div v-if="leaderboard.length" class="mt-8 bg-yellow-50 p-4 rounded-lg shadow-inner">
      <h3 class="text-xl font-bold text-yellow-800 mb-3">Leaderboard</h3>
      <ol class="list-decimal list-inside text-gray-700">
        <li v-for="(entry, index) in leaderboard" :key="index" class="mb-1">
          <span class="font-semibold">{{ entry.name }}</span> ({{ entry.category }}): {{ entry.score }}
        </li>
      </ol>
    </div>
  </div>
</template>