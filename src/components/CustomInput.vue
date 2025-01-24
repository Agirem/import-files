<script setup>
import { computed } from 'vue'

const props = defineProps({
  modelValue: {
    type: [String, Number],
    default: ''
  },
  label: {
    type: String,
    required: true
  },
  placeholder: {
    type: String,
    default: ''
  },
  type: {
    type: String,
    default: 'text'
  },
  maxLength: {
    type: Number,
    default: null
  },
  rows: {
    type: Number,
    default: 1
  },
  showCount: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['update:modelValue'])

const characterCount = computed(() => props.modelValue?.toString().length || 0)

const updateValue = (event) => {
  const value = event.target.value
  if (!props.maxLength || value.length <= props.maxLength) {
    emit('update:modelValue', value)
  }
}
</script>

<template>
  <div>
    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">{{ label }}</label>
    <template v-if="type === 'textarea'">
      <textarea
        :value="modelValue"
        @input="updateValue"
        :placeholder="placeholder"
        :rows="rows"
        :maxlength="maxLength"
        class="w-full px-3 py-2 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 text-gray-900 dark:text-white rounded-md focus:outline-none focus:ring-2 focus:ring-[#0F172A] dark:focus:ring-[#F8FAFC] placeholder-gray-400 dark:placeholder-gray-500"
      ></textarea>
      <div v-if="showCount" class="text-sm text-gray-600 dark:text-gray-400 mt-1">
        {{ characterCount }}/{{ maxLength }}
      </div>
    </template>
    <input
      v-else
      :type="type"
      :value="modelValue"
      @input="updateValue"
      :placeholder="placeholder"
      :maxlength="maxLength"
      class="w-full px-3 py-2 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 text-gray-900 dark:text-white rounded-md focus:outline-none focus:ring-2 focus:ring-[#0F172A] dark:focus:ring-[#F8FAFC] placeholder-gray-400 dark:placeholder-gray-500"
    >
  </div>
</template> 