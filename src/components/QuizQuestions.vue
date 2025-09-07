<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const categories = [
  {
    name: 'Science',
    questions: [
      {
        text: 'What planet is known as the Red Planet?',
        options: ['Earth', 'Mars', 'Jupiter', 'Venus'],
        answer: 'Mars',
        hint: 'It is the fourth planet from the Sun.',
        explanation: 'Mars is called the Red Planet because of its reddish appearance.'
      },
      {
        text: 'What is the chemical symbol for water?',
        options: ['O2', 'H2O', 'CO2', 'NaCl'],
        answer: 'H2O',
        hint: 'It contains hydrogen and oxygen.',
        explanation: 'H2O stands for two hydrogen atoms and one oxygen atom.'
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
        hint: 'He is on the one-dollar bill.',
        explanation: 'George Washington was the first President of the United States.'
      }
    ]
  }
]


const selectedCategory = ref(categories[0])
const currentQuestionIndex = ref(0)
const showHint = ref(false)
const showExplanation = ref(false)
const selectedOption = ref(null)

// Score tracking
const score = ref(0)

// Timer per question
const maxTime = 15
const timer = ref(maxTime)
let intervalId = null

// Leaderboard (local only)
const leaderboard = ref([])

const currentQuestion = computed(() => selectedCategory.value.questions[currentQuestionIndex.value])


function selectCategory(category) {
  selectedCategory.value = category
  currentQuestionIndex.value = 0
  showHint.value = false
  showExplanation.value = false
  selectedOption.value = null
  score.value = 0
  resetTimer()
}


function selectOption(option) {
  selectedOption.value = option
  showExplanation.value = true
  if (option === currentQuestion.value.answer && timer.value > 0) {
    score.value++
  }
  stopTimer()
}


function nextQuestion() {
  if (currentQuestionIndex.value < selectedCategory.value.questions.length - 1) {
    currentQuestionIndex.value++
    showHint.value = false
    showExplanation.value = false
    selectedOption.value = null
    resetTimer()
  } else {
    // End of quiz, add to leaderboard
    leaderboard.value.push({
      name: 'User',
      score: score.value,
      category: selectedCategory.value.name
    })
    // Sort leaderboard descending
    leaderboard.value.sort((a, b) => b.score - a.score)
  }
}

function resetTimer() {
  stopTimer()
  timer.value = maxTime
  intervalId = setInterval(() => {
    if (timer.value > 0) {
      timer.value--
    } else {
      showExplanation.value = true
      stopTimer()
    }
  }, 1000)
}

function stopTimer() {
  if (intervalId) {
    clearInterval(intervalId)
    intervalId = null
  }
}

onMounted(() => {
  resetTimer()
})

onUnmounted(() => {
  stopTimer()
})
</script>

<template>
  <div class="categories">
    <span v-for="cat in categories" :key="cat.name" :class="['category', {active: cat === selectedCategory}]" @click="selectCategory(cat)">
      {{ cat.name }}
    </span>
  </div>
  <div class="score-timer">
    <span><strong>Score:</strong> {{ score }}</span>
    <span class="timer"><strong>Time Left:</strong> {{ timer }}s</span>
  </div>
  <div class="question-block">
    <h2>Question:</h2>
    <p>{{ currentQuestion.text }}</p>
    <div class="options">
      <button v-for="opt in currentQuestion.options" :key="opt" :disabled="showExplanation" :class="['option', {selected: selectedOption === opt}]" @click="selectOption(opt)">
        {{ opt }}
      </button>
    </div>
    <button v-if="!showHint && !showExplanation" @click="showHint = true">Show Hint</button>
    <div v-if="showHint" class="hint">Hint: {{ currentQuestion.hint }}</div>
    <div v-if="showExplanation" class="explanation">
      <div v-if="selectedOption === currentQuestion.answer && timer > 0" style="color:green;">Correct!</div>
      <div v-else style="color:red;">Incorrect. The correct answer is {{ currentQuestion.answer }}.</div>
      <div>{{ currentQuestion.explanation }}</div>
      <button v-if="currentQuestionIndex < selectedCategory.questions.length - 1" @click="nextQuestion">Next Question</button>
      <div v-else><strong>Quiz Finished!</strong></div>
    </div>
  </div>
  <div class="leaderboard">
    <h3>Leaderboard</h3>
    <ol>
      <li v-for="entry in leaderboard" :key="entry.name + entry.category + entry.score">
        {{ entry.name }} ({{ entry.category }}): {{ entry.score }}
      </li>
    </ol>
  </div>
</template>

<style scoped>
.categories {
  margin-bottom: 1em;
}
.category {
  display: inline-block;
  margin-right: 1em;
  padding: 0.5em 1em;
  background: #eee;
  border-radius: 5px;
  cursor: pointer;
}
.category.active {
  background: #42b883;
  color: #fff;
}
.score-timer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1em;
  font-size: 1.1em;
}
.timer {
  background: #ffe082;
  padding: 0.3em 0.8em;
  border-radius: 8px;
}
.question-block {
  background: #f9f9f9;
  padding: 1em;
  border-radius: 8px;
  margin-bottom: 1em;
}
.leaderboard {
  background: #fffde7;
  padding: 1em;
  border-radius: 8px;
  margin-top: 2em;
}
.leaderboard ol {
  margin: 0;
  padding-left: 1.2em;
}
.options {
  margin: 1em 0;
}
.option {
  margin-right: 1em;
  margin-bottom: 0.5em;
  padding: 0.5em 1em;
  border: none;
  border-radius: 5px;
  background: #646cff;
  color: #fff;
  cursor: pointer;
}
.option.selected {
  background: #42b883;
}
.hint {
  margin-top: 1em;
  color: #888;
}
.explanation {
  margin-top: 1em;
  background: #e0f7fa;
  padding: 1em;
  border-radius: 5px;
}
</style>
