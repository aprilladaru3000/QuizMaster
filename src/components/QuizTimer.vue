<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// Props untuk konfigurasi dari komponen induk
const props = defineProps({
  // Durasi timer dalam detik
  duration: {
    type: Number,
    default: 30
  }
})

// Emits untuk berkomunikasi kembali ke komponen induk
const emit = defineEmits(['time-up'])

// State internal komponen
const timeLeft = ref(props.duration)
let timerInterval = null

// Computed property untuk menghitung persentase progress bar
const progress = computed(() => (timeLeft.value / props.duration) * 100)

// Fungsi untuk menghentikan timer
function stopTimer() {
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
  }
}

// Lifecycle hook: jalankan timer saat komponen dimuat
onMounted(() => {
  timerInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--
    } else {
      stopTimer()
      emit('time-up') // Kirim event saat waktu habis
    }
  }, 1000)
})

// Lifecycle hook: bersihkan timer saat komponen dihancurkan
onUnmounted(() => {
  stopTimer()
})
</script>

<template>
  <div class="w-full">
    <div class="text-center mb-2">
      <span class="text-2xl font-bold text-gray-700">{{ timeLeft }}</span>
      <span class="text-sm text-gray-500"> detik tersisa</span>
    </div>

    <div class="w-full bg-gray-200 rounded-full h-4 overflow-hidden">
      <div 
        class="bg-gradient-to-r from-green-400 to-blue-500 h-4 rounded-full transition-all duration-500 ease-linear"
        :style="{ width: progress + '%' }">
      </div>
    </div>
  </div>
</template>