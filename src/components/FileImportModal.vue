<script setup>
import { ref, computed } from 'vue'
import { animate } from '@oku-ui/motion'
import ModalHeader from './ModalHeader.vue'
import ModalFooter from './ModalFooter.vue'
import CustomInput from './CustomInput.vue'
import FileUploadZone from './FileUploadZone.vue'
import FileList from './FileList.vue'

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

const emit = defineEmits(['close'])

const fileName = ref('')
const description = ref('')
const uploadedFiles = ref([])

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

const handleFileDrop = (files) => {
  Array.from(files).forEach(file => {
    const fileType = getFileType(file.name)
    uploadedFiles.value.push({
      name: file.name,
      size: formatFileSize(file.size),
      totalSize: formatFileSize(file.size),
      progress: 0,
      status: fileType === 'word' ? 'error' : 'uploading',
      type: fileType,
      icon: getFileIcon(file.name)
    })
    
    if (fileType !== 'word') {
      simulateUploadProgress(uploadedFiles.value.length - 1)
    }
  })
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

const handleRetry = (index) => {
  const file = uploadedFiles.value[index]
  if (file.type !== 'word') {
    file.status = 'uploading'
    file.progress = 0
    simulateUploadProgress(index)
  }
}

const handleRemove = (index) => {
  uploadedFiles.value.splice(index, 1)
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
        <ModalHeader @close="$emit('close')" />

        <div class="p-4 space-y-4 overflow-y-auto custom-scrollbar flex-1">
          <CustomInput
            v-model="fileName"
            label="Nom du fichier"
            placeholder="Ex : Data_21_01_25"
          />

          <CustomInput
            v-model="description"
            label="Description"
            type="textarea"
            placeholder="Décrivez en quelques mots votre fichier"
            :rows="4"
            :maxLength="1000"
            :showCount="true"
          />

          <FileUploadZone
            :is-dark-mode="isDarkMode"
            @file-dropped="handleFileDrop"
          />

          <FileList
            :files="uploadedFiles"
            @retry="handleRetry"
            @remove="handleRemove"
          />
        </div>

        <ModalFooter
          @close="$emit('close')"
          @save="$emit('save')"
        />
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