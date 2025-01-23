<script setup>
import { ref, computed, nextTick } from 'vue'
import { animate } from '@oku-ui/motion'

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  isDarkMode: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['close', 'toggleTheme'])

const fileName = ref('')
const description = ref('')
const fileCount = ref(0)
const isDragging = ref(false)
const uploadedFiles = ref([])
const hoverStates = ref({})
const isHoveringDropZone = ref(false)
const draggedFileName = ref('')
const draggedFileType = ref('')

const characterCount = computed(() => description.value.length)

const updateDescription = (event) => {
  const text = event.target.value
  if (text.length <= 1000) {
    description.value = text
  }
}

const toggleTheme = () => {
  emit('toggleTheme', !props.isDarkMode)
}

const closeModal = () => {
  emit('close')
}

const removeFile = (index) => {
  uploadedFiles.value.splice(index, 1)
  fileCount.value = uploadedFiles.value.length
}

const shakeElement = async (el) => {
  await animate(el, 
    { x: [0, -10, 10, -10, 10, -5, 5, 0] }, 
    { 
      duration: 0.4,
      easing: [.36, .07, .19, .97]
    }
  )
}

const retryUpload = (event, index) => {
  const file = uploadedFiles.value[index]
  // Si c'est un fichier Word, on maintient l'erreur et on anime
  if (file.type === 'word') {
    file.status = 'error'
    // On remonte jusqu'à l'élément parent qui contient la classe file-item
    const fileElement = event.target.closest('.file-item')
    if (fileElement) {
      shakeElement(fileElement)
    }
    return
  }
  file.status = 'uploading'
  file.progress = 0
  simulateUploadProgress(index)
}

const handleFileDrop = (event) => {
  event.preventDefault()
  isDragging.value = false
  const files = event.dataTransfer?.files || event.target.files
  
  if (files.length > 0) {
    Array.from(files).forEach(file => {
      const fileType = getFileType(file.name)
      // Simuler un upload en cours
      uploadedFiles.value.push({
        name: file.name,
        size: formatFileSize(file.size),
        totalSize: formatFileSize(file.size),
        progress: 0,
        status: fileType === 'word' ? 'error' : 'uploading', // Simuler l'échec pour les fichiers Word
        type: fileType,
        icon: getFileIcon(file.name)
      })
      
      // Simuler la progression sauf pour les fichiers Word
      if (fileType !== 'word') {
        simulateUploadProgress(uploadedFiles.value.length - 1)
      }
    })
    
    fileCount.value = uploadedFiles.value.length
  }
}

const formatFileSize = (bytes) => {
  if (bytes === 0) return '0 B'
  const k = 1024
  const sizes = ['B', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return `${parseFloat((bytes / Math.pow(k, i)).toFixed(1))} ${sizes[i]}`
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
      return './src/assets/icons/excel.png'
    case 'json':
      return './src/assets/icons/json.png'
    case 'word':
      return './src/assets/icons/word.png'
    default:
      return null
  }
}

const simulateUploadProgress = (fileIndex) => {
  let progress = 0
  const interval = setInterval(() => {
    progress += Math.random() * 30
    if (progress >= 100) {
      progress = 100
      clearInterval(interval)
      uploadedFiles.value[fileIndex].status = 'completed'
      // Animation de complétion
      const element = document.querySelector(`[data-file-index="${fileIndex}"] .progress-circle`)
      if (element) {
        animate(element, {
          scale: [1, 1.1, 1],
          backgroundColor: ['#3B82F6', '#22C55E']
        }, {
          duration: 0.3
        })
      }
    }
    uploadedFiles.value[fileIndex].progress = Math.min(progress, 100)
  }, 500)
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

const setHoverState = (index, state) => {
  hoverStates.value[index] = state
}

const onModalEnter = async (el, done) => {
  await animate(el, { 
    opacity: [0, 1],
    scale: [0.95, 1],
  }, {
    duration: 0.2
  })
  done()
}

const onModalLeave = async (el, done) => {
  await animate(el, { 
    opacity: [1, 0],
    scale: [1, 0.95],
  }, {
    duration: 0.2
  })
  done()
}

const onFileEnter = async (el) => {
  await animate(el, {
    opacity: [0, 1],
    y: [20, 0]
  }, {
    duration: 0.3,
    delay: 0.1
  })
}

const onDropZoneEnter = async (el) => {
  await animate(el, {
    opacity: [0, 1],
    y: [10, 0]
  }, {
    duration: 0.2
  })
}
</script>

<template>
  <Transition
    :css="false"
    @enter="onModalEnter"
    @leave="onModalLeave"
  >
    <div v-if="isOpen" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
      <div class="bg-white dark:bg-[#0F172A] rounded-lg w-full max-w-xl flex flex-col max-h-[90vh] transition-colors duration-200">
        <!-- Header -->
        <div class="flex justify-between items-center p-4 border-b border-gray-200 dark:border-gray-700">
          <h2 class="text-xl font-semibold text-gray-900 dark:text-white">Processus d'importation</h2>
          <button @click="closeModal" class="text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-300">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>

        <!-- Content avec scroll -->
        <div class="p-4 space-y-4 overflow-y-auto custom-scrollbar flex-1">
          <!-- Nom du fichier -->
          <div>
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Nom du fichier</label>
            <input
              type="text"
              v-model="fileName"
              placeholder="Ex : Data_21_01_25"
              class="w-full px-3 py-2 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 text-gray-900 dark:text-white rounded-md focus:outline-none focus:ring-2 focus:ring-[#0F172A] dark:focus:ring-[#F8FAFC] placeholder-gray-400 dark:placeholder-gray-500"
            >
          </div>

          <!-- Description -->
          <div>
            <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Description</label>
            <textarea
              :value="description"
              @input="updateDescription"
              placeholder="Décrivez en quelques mots votre fichier"
              rows="4"
              :maxlength="1000"
              class="w-full px-3 py-2 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 text-gray-900 dark:text-white rounded-md focus:outline-none focus:ring-2 focus:ring-[#0F172A] dark:focus:ring-[#F8FAFC] placeholder-gray-400 dark:placeholder-gray-500"
            ></textarea>
            <!-- Compteur de caractères -->
            <div class="text-sm text-gray-600 dark:text-gray-400 mt-1">
              {{ characterCount }}/1000
            </div>
          </div>

          <!-- File upload area -->
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
                <svg width="30" height="36" viewBox="0 0 30 36" fill="none" xmlns="http://www.w3.org/2000/svg" class="mb-4">
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

            <!-- Liste des fichiers -->
            <div class="mt-4 space-y-2">
              <TransitionGroup
                :css="false"
                @enter="onFileEnter"
              >
                <div v-for="(file, index) in uploadedFiles" 
                     :key="file.name"
                     :data-file-id="file.name"
                     class="group flex items-center justify-between p-3 hover:bg-[#FAFAFA] dark:hover:bg-[#E2E8F0] rounded-lg transition-all duration-200 file-item">
                  <!-- Icône et nom du fichier -->
                  <div class="flex items-center space-x-3">
                    <img v-if="file.icon" :src="file.icon" :alt="file.type" class="w-8 h-8 object-contain">
                    <div class="flex flex-col">
                      <span class="text-sm font-medium text-gray-900 dark:text-white group-hover:dark:text-[#252F3F]">{{ file.name }}</span>
                      <span class="text-xs" :class="[
                        file.status === 'error' 
                          ? 'text-red-500' 
                          : 'text-gray-500 dark:text-gray-400 group-hover:dark:text-[#6B7280]'
                      ]">
                        {{ file.size }} / {{ file.totalSize }} - {{ file.status === 'error' ? 'Échec. Réessayer' : file.status === 'completed' ? 'Terminé' : `${Math.round(file.progress)}% complété` }}
                      </span>
                    </div>
                  </div>
                  
                  <!-- Actions -->
                  <div class="flex items-center space-x-4">
                    <!-- Loader circulaire ou pourcentage -->
                    <div v-if="file.status === 'uploading'"
                         @mouseenter="setHoverState(index, true)"
                         @mouseleave="setHoverState(index, false)"
                         class="relative w-8 h-8"
                         :data-file-index="index">
                      <div v-if="!hoverStates[index]" class="absolute inset-0 flex items-center justify-center">
                        <div class="w-8 h-8 rounded-full border-2 border-blue-500 dark:border-blue-400 flex items-center justify-center progress-circle">
                          <span class="text-xs font-medium text-gray-700 dark:text-gray-300">{{ Math.round(file.progress) }}%</span>
                        </div>
                      </div>
                      <button v-else
                              @click="(e) => retryUpload(e, index)"
                              class="absolute inset-0 flex items-center justify-center text-gray-400 dark:text-[#E2E8F0] hover:text-gray-600 dark:hover:text-gray-400">
                        <img src="../assets/icons/refresh.svg" alt="Réessayer" class="w-5 h-5">
                      </button>
                    </div>

                    <!-- Actions pour les fichiers en erreur -->
                    <div v-if="file.status === 'error'" class="flex items-center space-x-2">
                      <button @click="(e) => retryUpload(e, index)" class="p-1 text-gray-400 dark:text-[#E2E8F0] hover:text-gray-600 dark:hover:text-gray-400">
                        <img src="../assets/icons/refresh.svg" alt="Réessayer" class="w-5 h-5">
                      </button>
                      <button @click="removeFile(index)" class="p-1 text-gray-400 dark:text-[#E2E8F0] hover:text-gray-600 dark:hover:text-gray-400">
                        <img src="../assets/icons/delete.svg" alt="Supprimer" class="w-5 h-5">
                      </button>
                    </div>

                    <!-- Action pour les fichiers complétés -->
                    <button v-if="file.status === 'completed'"
                            @click="removeFile(index)" 
                            class="p-1 text-gray-400 dark:text-[#E2E8F0] hover:text-gray-600 dark:hover:text-gray-400">
                      <img src="../assets/icons/delete.svg" alt="Supprimer" class="w-5 h-5">
                    </button>
                  </div>
                </div>
              </TransitionGroup>
            </div>
          </div>
        </div>

        <!-- Footer -->
        <div class="flex justify-end space-x-3 p-4 border-t border-gray-200 dark:border-gray-700 mt-auto">
          <button
            @click="closeModal"
            class="px-4 py-2 text-sm font-medium text-slate-700 dark:text-slate-300 hover:text-slate-800 dark:hover:text-slate-200 border border-[#E2E8F0] dark:border-gray-600 rounded"
          >
            Annuler
          </button>
          <button
            class="px-4 py-2 text-sm font-medium text-white dark:text-[#1E293B] bg-slate-900 dark:bg-[#F1F5F9] rounded hover:bg-slate-800 dark:hover:bg-slate-700"
          >
            Enregistrer
          </button>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style scoped>
.custom-scrollbar {
  scrollbar-width: thin;
  scrollbar-color: #94A3B8 #F1F5F9;
}

.dark .custom-scrollbar {
  scrollbar-color: #475569 #1E293B;
}

.custom-scrollbar::-webkit-scrollbar {
  width: 8px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: #F1F5F9;
  border-radius: 4px;
}

.dark .custom-scrollbar::-webkit-scrollbar-track {
  background: #1E293B;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #94A3B8;
  border-radius: 4px;
  border: 2px solid #F1F5F9;
}

.dark .custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #475569;
  border: 2px solid #1E293B;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background-color: #64748B;
}

.dark .custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background-color: #334155;
}
</style>