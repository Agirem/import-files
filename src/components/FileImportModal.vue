<script setup>
import { ref } from 'vue'

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['close'])

const fileName = ref('')
const description = ref('')
const fileCount = ref(0)
const isDragging = ref(false)
const uploadedFiles = ref([])

const closeModal = () => {
  emit('close')
}

const removeFile = (index) => {
  uploadedFiles.value.splice(index, 1)
  fileCount.value = uploadedFiles.value.length
}

const handleFileDrop = (event) => {
  event.preventDefault()
  isDragging.value = false
  const files = event.dataTransfer?.files || event.target.files
  
  if (files.length > 0) {
    Array.from(files).forEach(file => {
      // Simuler un upload en cours
      uploadedFiles.value.push({
        name: file.name,
        size: formatFileSize(file.size),
        totalSize: formatFileSize(file.size),
        progress: 0,
        status: 'uploading',
        type: getFileType(file.name),
        icon: getFileIcon(file.name)
      })
      
      // Simuler la progression
      simulateUploadProgress(uploadedFiles.value.length - 1)
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
    }
    uploadedFiles.value[fileIndex].progress = Math.min(progress, 100)
  }, 500)
}

const preventDefault = (e) => {
  e.preventDefault()
  if (e.type === 'dragenter') {
    isDragging.value = true
  }
}

const handleDragLeave = (e) => {
  e.preventDefault()
  isDragging.value = false
}
</script>

<template>
  <div v-if="isOpen" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
    <div class="bg-white rounded-lg w-full max-w-md">
      <!-- Header -->
      <div class="flex justify-between items-center p-4 border-b">
        <h2 class="text-xl font-semibold">Processus d'importation</h2>
        <button @click="closeModal" class="text-gray-500 hover:text-gray-700">
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- Content -->
      <div class="p-4 space-y-4">
        <!-- Nom du fichier -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Nom du fichier</label>
          <input
            type="text"
            v-model="fileName"
            placeholder="Ex : Data_21_01_25"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
        </div>

        <!-- Description -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
          <textarea
            v-model="description"
            placeholder="Décrivez en quelques mots votre fichier"
            rows="4"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          ></textarea>
        </div>

        <!-- File counter -->
        <div class="text-sm text-gray-600">
          {{ fileCount }}/1000
        </div>

        <!-- File upload area -->
        <div>
          <h3 class="text-sm font-medium text-slate-900">Importez un fichier</h3>
          <p class="text-sm text-slate-500">Vous pouvez ajoutez plus de 10 fichiers.</p>
          <div
            @drop="handleFileDrop"
            @dragover="preventDefault"
            @dragenter="preventDefault"
            @dragleave="handleDragLeave"
            class="mt-2 border border-dashed rounded-lg p-8 text-center transition-colors"
            :class="[
              isDragging ? 'border-[#EFF6FF] bg-[#93C5FD]' : 'border-[#CBD5E1] bg-[#F8FAFC]'
            ]"
          >
            <div class="flex flex-col items-center">
              <svg width="30" height="36" viewBox="0 0 30 36" fill="none" xmlns="http://www.w3.org/2000/svg" class="mb-4">
                <path fill-rule="evenodd" clip-rule="evenodd" d="M24.4388 3.19609C24.8132 3.22199 25.187 3.25043 25.56 3.28139C28.1255 3.49432 30.0001 5.67135 30.0001 8.18035V24.25C30.0001 27.0114 27.7615 29.25 25.0001 29.25V25.5C25.0001 15.6301 17.3741 7.5414 7.69287 6.80469C8.23315 4.91878 9.85942 3.45409 11.9402 3.28139C12.3132 3.25043 12.6869 3.22199 13.0613 3.19609C13.8945 1.59411 15.5695 0.5 17.5001 0.5H20.0001C21.9306 0.5 23.6056 1.59411 24.4388 3.19609ZM15.0001 5.5C15.0001 4.11929 16.1194 3 17.5001 3H20.0001C21.3808 3 22.5001 4.11929 22.5001 5.5H15.0001Z" fill="#1E293B"/>
                <path d="M0 12.375C0 10.6491 1.39911 9.25 3.125 9.25H3.75C7.20178 9.25 10 12.0482 10 15.5V18.625C10 20.3509 11.3991 21.75 13.125 21.75H16.25C19.7018 21.75 22.5 24.5482 22.5 28V32.375C22.5 34.1009 21.1009 35.5 19.375 35.5H3.125C1.39911 35.5 0 34.1009 0 32.375V12.375Z" fill="#1E293B"/>
                <path d="M12.5 15.5C12.5 13.3116 11.6966 11.3108 10.3687 9.77651C16.0335 11.2563 20.4937 15.7165 21.9735 21.3813C20.4392 20.0534 18.4384 19.25 16.25 19.25H13.125C12.7798 19.25 12.5 18.9702 12.5 18.625V15.5Z" fill="#1E293B"/>
              </svg>
              <p class="text-sm font-medium text-slate-900 mb-1">Choisissez un fichier ou glissez et deposez</p>
              <p class="text-xs text-slate-500">CSV, EXCEL(xls,xlsx..), JSON</p>
              <p class="text-xs text-slate-500 mt-2">50 MB maximum</p>
            </div>
          </div>

          <!-- Liste des fichiers -->
          <div class="mt-4 space-y-2">
            <div v-for="(file, index) in uploadedFiles" :key="index" 
                 class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
              <!-- Icône et nom du fichier -->
              <div class="flex items-center space-x-3">
                <img v-if="file.icon" :src="file.icon" :alt="file.type" class="w-8 h-8 object-contain">
                <span class="text-sm font-medium">{{ file.name }}</span>
              </div>
              
              <!-- Taille et progression -->
              <div class="flex items-center space-x-4">
                <span class="text-xs text-gray-500">
                  {{ file.size }} / {{ file.totalSize }}
                </span>
                
                <!-- Barre de progression -->
                <div class="w-24 bg-gray-200 rounded-full h-1.5">
                  <div class="bg-blue-600 h-1.5 rounded-full" 
                       :style="{ width: file.progress + '%' }"></div>
                </div>

                <!-- Bouton de suppression -->
                <button @click="removeFile(index)" 
                        class="text-gray-400 hover:text-gray-600">
                  <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                  </svg>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Footer -->
      <div class="flex justify-end space-x-3 p-4 border-t">
        <button
          @click="closeModal"
          class="px-4 py-2 text-sm font-medium text-slate-700 hover:text-slate-800 border border-[#E2E8F0] rounded"
        >
          Annuler
        </button>
        <button
          class="px-4 py-2 text-sm font-medium text-white bg-slate-900 rounded hover:bg-slate-800"
        >
          Enregistrer
        </button>
      </div>
    </div>
  </div>
</template>