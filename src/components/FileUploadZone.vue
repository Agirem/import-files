<script setup>
import { ref } from 'vue'
import { animate } from 'motion'

const props = defineProps({
  isDarkMode: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['fileDropped'])

const isDragging = ref(false)
const draggedFileName = ref('')
const draggedFileType = ref('')

const onDropZoneEnter = async (el) => {
  await animate(el, {
    opacity: [0, 1],
    y: [20, 0],
    scale: [0.95, 1]
  }, {
    duration: 0.3,
    easing: [0.25, 0.1, 0.25, 1]
  })
}

const getFileType = (filename) => {
  const ext = filename.split('.').pop().toLowerCase()
  if (['xls', 'xlsx'].includes(ext)) return 'excel'
  if (ext === 'json') return 'json'
  if (ext === 'csv') return 'csv'
  if (['doc', 'docx'].includes(ext)) return 'word'
  return 'unknown'
}

const getFileIcon = (filename) => {
  const type = getFileType(filename)
  switch (type) {
    case 'excel':
      return new URL('../assets/icons/excel.png', import.meta.url).href
    case 'json':
      return new URL('../assets/icons/json.png', import.meta.url).href
    case 'word':
      return new URL('../assets/icons/word.png', import.meta.url).href
    default:
      return null
  }
}

const preventDefault = (e) => {
  e.preventDefault()
  if (e.type === 'dragenter') {
    isDragging.value = true
    if (e.dataTransfer.items && e.dataTransfer.items[0]) {
      const file = e.dataTransfer.items[0].getAsFile()
      if (file) {
        draggedFileName.value = file.name
        draggedFileType.value = getFileType(file.name)
      }
    } else if (e.dataTransfer.files && e.dataTransfer.files[0]) {
      const file = e.dataTransfer.files[0]
      draggedFileName.value = file.name
      draggedFileType.value = getFileType(file.name)
    }
  }
}

const handleDragLeave = (e) => {
  e.preventDefault()
  isDragging.value = false
  draggedFileName.value = ''
  draggedFileType.value = ''
}

const handleFileDrop = (event) => {
  event.preventDefault()
  isDragging.value = false
  const files = event.dataTransfer?.files || event.target.files
  if (files.length > 0) {
    emit('fileDropped', files)
  }
}
</script>

<template>
  <div>
    <h3 class="text-sm font-medium text-slate-900 dark:text-white">Importez un fichier</h3>
    <div class="flex items-center text-sm text-slate-500 dark:text-slate-400 mt-1 space-x-4">
      <p>Vous pouvez ajouter plus de 10 fichiers.</p>
    </div>
    <div
      @drop="handleFileDrop"
      @dragover="preventDefault"
      @dragenter="preventDefault"
      @dragleave="handleDragLeave"
      class="mt-2 border border-dashed rounded-lg p-8 text-center transition-colors relative"
      :class="[
        isDragging 
          ? 'border-[#EFF6FF] bg-[#93C5FD] dark:border-blue-500 dark:bg-blue-900' 
          : 'border-[#CBD5E1] dark:border-gray-600 bg-[#F8FAFC] dark:bg-gray-800'
      ]"
    >
      <div class="flex flex-col items-center" :class="{ 'opacity-0': isDragging }">
        <svg width="30" height="36" viewBox="0 0 30 36" fill="none" xmlns="http://www.w3.org/2000/svg" 
             class="mb-4 transform-origin-top hover:animate-paper-wave transition-transform duration-300">
          <path fill-rule="evenodd" clip-rule="evenodd" d="M24.4388 3.19609C24.8132 3.22199 25.187 3.25043 25.56 3.28139C28.1255 3.49432 30.0001 5.67135 30.0001 8.18035V24.25C30.0001 27.0114 27.7615 29.25 25.0001 29.25V25.5C25.0001 15.6301 17.3741 7.5414 7.69287 6.80469C8.23315 4.91878 9.85942 3.45409 11.9402 3.28139C12.3132 3.25043 12.6869 3.22199 13.0613 3.19609C13.8945 1.59411 15.5695 0.5 17.5001 0.5H20.0001C21.9306 0.5 23.6056 1.59411 24.4388 3.19609ZM15.0001 5.5C15.0001 4.11929 16.1194 3 17.5001 3H20.0001C21.3808 3 22.5001 4.11929 22.5001 5.5H15.0001Z" :fill="isDarkMode ? '#F8FAFC' : '#1E293B'"/>
          <path d="M0 12.375C0 10.6491 1.39911 9.25 3.125 9.25H3.75C7.20178 9.25 10 12.0482 10 15.5V18.625C10 20.3509 11.3991 21.75 13.125 21.75H16.25C19.7018 21.75 22.5 24.5482 22.5 28V32.375C22.5 34.1009 21.1009 35.5 19.375 35.5H3.125C1.39911 35.5 0 34.1009 0 32.375V12.375Z" :fill="isDarkMode ? '#F8FAFC' : '#1E293B'"/>
          <path d="M12.5 15.5C12.5 13.3116 11.6966 11.3108 10.3687 9.77651C16.0335 11.2563 20.4937 15.7165 21.9735 21.3813C20.4392 20.0534 18.4384 19.25 16.25 19.25H13.125C12.7798 19.25 12.5 18.9702 12.5 18.625V15.5Z" :fill="isDarkMode ? '#F8FAFC' : '#1E293B'"/>
        </svg>
        <p class="text-sm font-medium text-slate-900 dark:text-[#E2E8F0] mb-1">Choisissez un fichier ou glissez et deposez</p>
        <p class="text-xs text-slate-500 dark:text-[#E2E8F0]">CSV, EXCEL(xls,xlsx..), JSON</p>
        <p class="text-xs text-slate-500 dark:text-[#E2E8F0] mt-2">50 MB maximum</p>
      </div>

      <!-- Affichage du fichier en cours de drag -->
      <Transition
        :css="false"
        @enter="onDropZoneEnter"
      >
        <div v-if="isDragging && draggedFileName" 
             class="absolute inset-0 flex items-center justify-center bg-[#1E293B] dark:bg-gray-800 rounded-lg transition-all duration-200">
          <div class="flex items-center space-x-2 px-3 py-1.5">
            <img v-if="getFileIcon(draggedFileName)" :src="getFileIcon(draggedFileName)" alt="File icon" class="w-5 h-5">
            <span class="text-sm text-white dark:text-[#E2E8F0]">{{ draggedFileName }}</span>
          </div>
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
@keyframes paper-wave {
  0% {
    transform: rotate(0deg) translateY(0);
  }
  25% {
    transform: rotate(-5deg) translateY(-2px);
  }
  50% {
    transform: rotate(0deg) translateY(-4px);
  }
  75% {
    transform: rotate(5deg) translateY(-2px);
  }
  100% {
    transform: rotate(0deg) translateY(0);
  }
}

.transform-origin-top {
  transform-origin: top center;
}

.hover\:animate-paper-wave:hover {
  animation: paper-wave 2s ease-in-out infinite;
}
</style> 