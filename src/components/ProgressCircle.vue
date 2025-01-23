<script setup>
import { ref } from 'vue'
import { animate } from 'motion'

const props = defineProps({
  progress: {
    type: Number,
    required: true
  },
  index: {
    type: Number,
    required: true
  },
  onCancel: {
    type: Function,
    required: true
  }
})

const isHovered = ref(false)

const onComplete = async (el) => {
  await animate(el, {
    scale: [1, 1.2, 1],
    backgroundColor: ['#3B82F6', '#22C55E']
  }, {
    duration: 0.4,
    easing: [0.25, 0.1, 0.25, 1]
  })
}
</script>

<template>
  <div
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
    class="relative w-8 h-8"
  >
    <div v-if="!isHovered" class="absolute inset-0 flex items-center justify-center">
      <div class="w-8 h-8 rounded-full border-2 border-blue-500 dark:border-blue-400 flex items-center justify-center progress-circle">
        <span class="text-xs font-medium text-gray-700 dark:text-gray-300">{{ Math.round(progress) }}%</span>
      </div>
    </div>
    <button v-else
            @click="onCancel"
            class="absolute inset-0 flex items-center justify-center text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-300">
      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
      </svg>
    </button>
  </div>
</template> 