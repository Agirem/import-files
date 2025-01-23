<script setup>
import { animate } from 'motion'
import ProgressCircle from './ProgressCircle.vue'
import refreshIcon from '../assets/icons/refresh.svg'
import deleteIcon from '../assets/icons/delete.svg'

const props = defineProps({
  file: {
    type: Object,
    required: true
  },
  index: {
    type: Number,
    required: true
  }
})

const emit = defineEmits(['retry', 'remove'])

const shakeElement = async (el) => {
  // Animation combinée avec motion
  await animate(el, 
    {
      x: [0, -35, 35, -35, 35, -20, 20, -10, 10, 0],
      rotate: [0, -6, 6, -6, 6, -3, 3, -1, 1, 0],
      border: ['none', '2px solid rgba(239, 68, 68, 1)', '2px solid rgba(239, 68, 68, 1)', '2px solid rgba(239, 68, 68, 0)'],
      backgroundColor: ['transparent', 'rgba(239, 68, 68, 0.1)', 'rgba(239, 68, 68, 0.1)', 'transparent']
    },
    {
      duration: 0.8,
      easing: [0.25, 0.1, 0.25, 1],
      repeat: 1
    }
  )
}

const handleRetry = (event) => {
  if (props.file.type === 'word') {
    const fileElement = event.target.closest('.file-item')
    if (fileElement) {
      shakeElement(fileElement)
    }
  }
  emit('retry', props.index)
}
</script>

<template>
  <div 
    :data-file-id="file.name"
    class="group flex items-center justify-between p-3 hover:bg-[#FAFAFA] dark:hover:bg-[#E2E8F0] rounded-lg transition-all duration-500 file-item"
  >
    <!-- Icône et nom du fichier -->
    <div class="flex items-center space-x-3">
      <img v-if="file.icon" :src="file.icon" :alt="file.type" class="w-8 h-8 object-contain">
      <div class="flex flex-col">
        <span class="text-sm font-medium text-gray-900 dark:text-white group-hover:dark:text-[#252F3F]">{{ file.name }}</span>
        <span class="text-xs" :class="[
          file.status === 'error' 
            ? 'text-red-500 dark:text-red-500' 
            : 'text-gray-500 dark:text-gray-400 group-hover:dark:text-[#6B7280]'
        ]">
          {{ file.size }} / {{ file.totalSize }} - {{ file.status === 'error' ? 'Échec. Réessayer' : file.status === 'completed' ? 'Terminé' : `${Math.round(file.progress)}% complété` }}
        </span>
      </div>
    </div>
    
    <!-- Actions -->
    <div class="flex items-center space-x-4">
      <!-- Loader circulaire ou pourcentage -->
      <ProgressCircle
        v-if="file.status === 'uploading'"
        :progress="file.progress"
        :index="index"
        @cancel="$emit('remove', index)"
      />

      <!-- Actions pour les fichiers en erreur -->
      <div v-if="file.status === 'error'" class="flex items-center space-x-2">
        <button @click="handleRetry" class="p-1 text-gray-400 dark:text-[#E2E8F0] hover:text-gray-600 dark:hover:text-gray-400 transition-transform hover:rotate-180 duration-300">
          <img :src="refreshIcon" alt="Réessayer" class="w-5 h-5">
        </button>
        <button @click="$emit('remove', index)" class="p-1 text-gray-400 dark:text-[#E2E8F0] hover:text-gray-600 dark:hover:text-gray-400 hover:animate-shake">
          <img :src="deleteIcon" alt="Supprimer" class="w-5 h-5">
        </button>
      </div>

      <!-- Action pour les fichiers complétés -->
      <button v-if="file.status === 'completed'"
              @click="$emit('remove', index)" 
              class="p-1 text-gray-400 dark:text-[#E2E8F0] hover:text-gray-600 dark:hover:text-gray-400 hover:animate-shake">
        <img :src="deleteIcon" alt="Supprimer" class="w-5 h-5">
      </button>
    </div>
  </div>
</template>

<style scoped>
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20% { transform: translateX(-2px) rotate(-2deg); }
  40% { transform: translateX(2px) rotate(2deg); }
  60% { transform: translateX(-2px) rotate(-1deg); }
  80% { transform: translateX(2px) rotate(1deg); }
}

.hover\:animate-shake:hover {
  animation: shake 0.5s ease-in-out;
}
</style> 