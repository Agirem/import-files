<script setup>
import { animate } from 'motion'
import FileItem from './FileItem.vue'

const props = defineProps({
  files: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['retry', 'remove'])

const onFileEnter = async (el) => {
  await animate(
    el,
    {
      opacity: [0, 1],
      y: [50, 0],
      scale: [0.9, 1]
    },
    {
      duration: 0.5,
      easing: [.22, .03, .26, 1]
    }
  )
}
</script>

<template>
  <div class="mt-4 space-y-2">
    <TransitionGroup
      :css="false"
      @enter="onFileEnter"
      tag="div"
      class="space-y-2"
    >
      <FileItem
        v-for="(file, index) in files"
        :key="file.name"
        :file="file"
        :index="index"
        @retry="$emit('retry', $event)"
        @remove="$emit('remove', $event)"
      />
    </TransitionGroup>
  </div>
</template> 