<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
    <div class="max-w-6xl mx-auto">
      <!-- Header -->
      <header class="text-center mb-8">
        <a href="https://github.com/NuclearMissile/vue-typing-practice" target="_blank" rel="noreferrer noopener">
          <h1 class="text-4xl font-bold text-gray-800 mb-2">Typing Practice</h1>
        </a>
        <p class="text-gray-600">Improve your typing speed and accuracy</p>
      </header>

      <!-- Stats Dashboard -->
      <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
        <div class="bg-white rounded-lg shadow-md p-4 text-center">
          <div class="flex items-center justify-center mb-2">
            <Zap class="w-5 h-5 text-yellow-500 mr-2"/>
            <span class="text-sm font-medium text-gray-600">WPM</span>
          </div>
          <div class="text-2xl font-bold text-gray-800">{{ wpm }}</div>
        </div>

        <div class="bg-white rounded-lg shadow-md p-4 text-center">
          <div class="flex items-center justify-center mb-2">
            <Target class="w-5 h-5 text-green-500 mr-2"/>
            <span class="text-sm font-medium text-gray-600">Accuracy</span>
          </div>
          <div class="text-2xl font-bold text-gray-800">{{ accuracy }}%</div>
        </div>

        <div class="bg-white rounded-lg shadow-md p-4 text-center">
          <div class="flex items-center justify-center mb-2">
            <Clock class="w-5 h-5 text-blue-500 mr-2"/>
            <span class="text-sm font-medium text-gray-600">Time</span>
          </div>
          <div class="text-2xl font-bold text-gray-800">{{ formatTime(timeElapsed) }}</div>
        </div>

        <div class="bg-white rounded-lg shadow-md p-4 text-center">
          <div class="flex items-center justify-center mb-2">
            <X class="w-5 h-5 text-red-500 mr-2"/>
            <span class="text-sm font-medium text-gray-600">Errors</span>
          </div>
          <div class="text-2xl font-bold text-gray-800">{{ errors }}</div>
        </div>
      </div>

      <!-- Main Typing Area -->
      <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
        <div class="mb-6">
          <div
              ref="textDisplayRef"
              class="bg-gray-50 rounded-lg p-6 border-2 border-gray-200 min-h-40 leading-relaxed focus:border-blue-500 focus:outline-none"
              tabindex="0"
              @keydown="handleKeyDown"
          >
            <span
                v-for="(char, index) in currentText.split('')"
                :key="index"
                :class="getCharClass(char, index)"
            >
              {{ char }}
            </span>
          </div>
        </div>

        <!-- Progress Bar -->
        <div class="mb-4">
          <div class="flex justify-between text-sm text-gray-600 mb-1">
            <span>Progress</span>
            <span>{{ Math.round((userInput.length / currentText.length) * 100) }}%</span>
          </div>
          <div class="w-full bg-gray-200 rounded-full h-2">
            <div
                class="bg-blue-500 h-2 rounded-full transition-all duration-300"
                :style="{ width: `${(userInput.length / currentText.length) * 100}%` }"
            ></div>
          </div>
        </div>

        <!-- Control Buttons -->
        <div class="flex flex-wrap gap-3 justify-center">
          <button
              @click="togglePause"
              :disabled="status === 'waiting' || status === 'completed'"
              class="flex items-center px-4 py-2 bg-yellow-500 text-white rounded-lg hover:bg-yellow-600 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
          >
            <Play v-if="status === 'paused'" class="w-4 h-4 mr-2"/>
            <Pause v-else class="w-4 h-4 mr-2"/>
            {{ status === 'paused' ? 'Resume' : 'Pause' }}
          </button>

          <button
              @click="resetTest"
              class="flex items-center px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors"
          >
            <RotateCcw class="w-4 h-4 mr-2"/>
            Reset
          </button>

          <button
              @click="newText"
              class="flex items-center px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600 transition-colors"
          >
            <ListRestart class="w-4 h-4 mr-2"/>
            New Text
          </button>

          <button
              @click="showSettings = !showSettings"
              class="flex items-center px-4 py-2 bg-gray-500 text-white rounded-lg hover:bg-gray-600 transition-colors"
          >
            <Settings class="w-4 h-4 mr-2"/>
            Settings
          </button>
        </div>
      </div>

      <!-- Settings Panel -->
      <div v-if="showSettings" class="bg-white rounded-lg shadow-lg p-6 mb-6">
        <h3 class="text-xl font-bold text-gray-800 mb-4">Settings</h3>
        <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Difficulty
            </label>
            <select
                v-model="difficulty"
                class="w-full p-2 border border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none"
            >
              <option
                  v-for="diff in Object.keys(sampleTexts)"
                  :key="diff"
                  :value="diff"
              >
                {{ diff }}
              </option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Backspace
            </label>
            <select
                v-model="backspaceMode"
                class="w-full p-2 border border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none"
            >
              <option value="enable">enable</option>
              <option value="disable">disable</option>
            </select>
          </div>
        </div>
      </div>

      <!-- Completion Modal -->
      <div
          v-if="status === 'completed'"
          class="fixed inset-0 bg-gray-500/70 flex items-center justify-center z-50"
      >
        <div class="bg-white rounded-lg p-8 max-w-md w-full mx-4">
          <h2 class="text-2xl font-bold text-gray-800 mb-4 text-center">
            🎉 Congratulations!
          </h2>
          <div class="space-y-3 mb-6">
            <div class="flex justify-between">
              <span class="text-gray-600">Words Per Minute:</span>
              <span class="font-bold text-blue-600">{{ wpm }} WPM</span>
            </div>
            <div class="flex justify-between">
              <span class="text-gray-600">Accuracy:</span>
              <span class="font-bold text-green-600">{{ accuracy }}%</span>
            </div>
            <div class="flex justify-between">
              <span class="text-gray-600">Time Taken:</span>
              <span class="font-bold text-purple-600">{{ formatTime(timeElapsed) }}</span>
            </div>
            <div class="flex justify-between">
              <span class="text-gray-600">Errors:</span>
              <span class="font-bold text-red-600">{{ errors }}</span>
            </div>
          </div>
          <div class="flex gap-3">
            <button
                @click="resetTest"
                class="flex-1 px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors"
            >
              Try Again
            </button>
            <button
                @click="newText"
                class="flex-1 px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600 transition-colors"
            >
              New Text
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {computed, nextTick, onMounted, onUnmounted, ref, watch} from 'vue'
import {Clock, ListRestart, Pause, Play, RotateCcw, Settings, Target, X, Zap} from 'lucide-vue-next'

const sampleTexts = {
  easy: [
    "The quick brown fox jumps over the lazy dog.",
    "Typing practice improves your speed and accuracy over time.",
    "React is a JavaScript library for building user interfaces.",
    "Practice makes perfect, especially when it comes to typing skills.",
    "Learning to type without looking at the keyboard is called touch typing.",
    "Simple sentences help beginners develop basic typing skills.",
    "Regular practice is the key to becoming a proficient typist.",
    "Good posture is important when typing for extended periods.",
    "Typing is an essential skill in today's digital world.",
    "Focus on accuracy first, then gradually increase your speed."
  ],
  medium: [
    "Programming is the art of telling another human what one wants the computer to do.",
    "The best way to predict the future is to invent it. Alan Kay said that.",
    "Typing quickly and accurately is an essential skill for modern computer users.",
    "JavaScript is a high-level, often just-in-time compiled language that conforms to the ECMAScript specification.",
    "React uses a virtual DOM to efficiently update the UI when the underlying data changes.",
    "Efficient typing can significantly increase your productivity in various computer-related tasks.",
    "The QWERTY keyboard layout was designed in the 1870s for mechanical typewriters to prevent jamming.",
    "Touch typing involves placing your fingers on the home row keys and typing without looking at the keyboard.",
    "Modern keyboards have evolved from mechanical typewriters but maintain the same basic layout and functionality.",
    "Learning keyboard shortcuts can save time and increase efficiency when working with computer applications."
  ],
  hard: [
    "The journey of a thousand miles begins with a single step. Typing efficiently is similar - it starts with learning the correct finger positions.",
    "In computer science, artificial intelligence, sometimes called machine intelligence, is intelligence demonstrated by machines, unlike the natural intelligence displayed by humans.",
    "The World Wide Web (WWW), commonly known as the Web, is an information system where documents and other web resources are identified by Uniform Resource Locators, which may be interlinked by hypertext.",
    "Quantum computing is the use of quantum-mechanical phenomena such as superposition and entanglement to perform computation. Computers that perform quantum computations are known as quantum computers.",
    "The process of developing a structured set of instructions to be followed by a computer or electronic device to perform a specific task or solve a particular problem is called computer programming.",
    "The development of typing skills requires consistent practice and attention to technique, including proper finger placement, posture, and rhythmic keystroke patterns that maximize efficiency and minimize strain.",
    "Neuroplasticity, the brain's ability to reorganize itself by forming new neural connections, plays a crucial role in learning complex motor skills such as touch typing, allowing for improved speed and accuracy over time.",
    "The evolution of human-computer interaction has been significantly influenced by advancements in input methods, from punch cards and mechanical keyboards to touchscreens and voice recognition, each requiring different skill sets and adaptations.",
    "Professional typists often achieve speeds exceeding 100 words per minute through a combination of muscle memory, anticipatory reading, and specialized techniques that minimize unnecessary finger movement and maximize keystroke efficiency.",
    "The integration of ergonomic principles into keyboard design aims to reduce the risk of repetitive strain injuries by promoting natural wrist and hand positions, appropriate key resistance, and optimal key spacing for different hand sizes."
  ]
}

// Reactive state
const userInput = ref('')
const timeElapsed = ref(0)
const errors = ref(0)
const status = ref('waiting') // waiting, playing, paused, completed
const showSettings = ref(false)
const backspaceMode = ref('enable')
const difficulty = ref('easy')
const currentText = ref('')
const currentIndex = ref(0)

// Refs
const textDisplayRef = ref(null)
let timerInterval = null

// Computed properties
const wpm = computed(() => {
  const timeInMinutes = timeElapsed.value / 60
  const wordsTyped = userInput.value.trim().split(' ').length
  return timeInMinutes > 0 ? Math.round(wordsTyped / timeInMinutes) : 0
})

const accuracy = computed(() => {
  const charactersTyped = userInput.value.length
  return charactersTyped > 0 ? Math.round(((charactersTyped - errors.value) / charactersTyped) * 100) : 0
})

// Methods
const getRandomText = (diff) => {
  let ret = sampleTexts[diff][Math.floor(Math.random() * sampleTexts[diff].length)]
  while (ret === currentText.value && sampleTexts[diff].length > 1) {
    ret = sampleTexts[diff][Math.floor(Math.random() * sampleTexts[diff].length)]
  }
  return ret
}

const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = Math.floor(seconds % 60)
  return `${mins}:${secs.toString().padStart(2, '0')}`
}

const getCharClass = (char, index) => {
  let className = 'text-lg '

  if (index < userInput.value.length) {
    if (userInput.value[index] === char) {
      className += 'text-green-600 bg-green-100'
    } else {
      className += 'text-red-600 bg-red-100'
    }
  } else if (index === currentIndex.value) {
    className += 'text-gray-800 bg-blue-200 animate-pulse'
  } else {
    className += 'text-gray-600'
  }

  return className
}

const handleKeyDown = (e) => {
  if (status.value === 'paused' || status.value === 'completed') {
    return
  }

  if (e.key.length === 1 || (e.key === 'Backspace' && userInput.value.length > 0 && backspaceMode.value === 'enable')) {
    e.preventDefault()

    if (status.value === 'waiting') {
      status.value = 'playing'
    }

    let newInput = userInput.value
    if (e.key === 'Backspace') {
      newInput = userInput.value.slice(0, -1)
    } else if (status.value !== 'completed' && currentText.value.length > userInput.value.length) {
      newInput = userInput.value + e.key
    }

    userInput.value = newInput
    currentIndex.value = newInput.length

    let errorCount = 0
    for (let i = 0; i < newInput.length; i++) {
      if (newInput[i] !== currentText.value[i]) {
        errorCount++
      }
    }
    errors.value = errorCount

    if (newInput.length === currentText.value.length) {
      status.value = 'completed'
    }
  }
}

const resetTest = () => {
  userInput.value = ''
  timeElapsed.value = 0
  errors.value = 0
  status.value = 'waiting'
  currentIndex.value = 0
  nextTick(() => {
    if (textDisplayRef.value) {
      textDisplayRef.value.focus()
    }
  })
}

const newText = () => {
  currentText.value = getRandomText(difficulty.value)
  resetTest()
}

const togglePause = () => {
  if (status.value === 'playing') {
    status.value = 'paused'
  } else if (status.value === 'paused') {
    status.value = 'playing'
  }
}

// Watchers
watch(status, (newStatus) => {
  if (newStatus === 'playing') {
    timerInterval = setInterval(() => {
      timeElapsed.value += 0.5
    }, 500)
  } else {
    if (timerInterval) {
      clearInterval(timerInterval)
      timerInterval = null
    }
  }

  if (newStatus === 'playing' || newStatus === 'waiting') {
    nextTick(() => {
      if (textDisplayRef.value) {
        textDisplayRef.value.focus()
      }
    })
  }
})

watch(difficulty, () => {
  currentText.value = getRandomText(difficulty.value)
  resetTest()
})

watch(backspaceMode, () => {
  nextTick(() => {
    if (textDisplayRef.value) {
      textDisplayRef.value.focus()
    }
  })
})

// Lifecycle hooks
onMounted(() => {
  if (textDisplayRef.value) {
    textDisplayRef.value.focus()
  }
  currentText.value = getRandomText(difficulty.value)
})

onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval)
  }
})
</script>