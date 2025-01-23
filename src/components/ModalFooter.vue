<script setup>
defineEmits(['close', 'save'])
</script>

<template>
  <div class="flex justify-end space-x-3 p-4 border-t border-gray-200 dark:border-gray-700 mt-auto">
    <button
      @click="$emit('close')"
      class="px-4 py-2 text-sm font-medium text-slate-700 dark:text-slate-300 hover:text-slate-800 dark:hover:text-slate-200 border border-[#E2E8F0] dark:border-gray-600 rounded"
    >
      Annuler
    </button>
    <button
      @click="$emit('save')"
      class="px-4 py-2 text-sm font-medium text-white dark:text-[#1E293B] bg-slate-900 dark:bg-[#F1F5F9] rounded hover:bg-slate-800 dark:hover:bg-slate-700"
    >
      Enregistrer
    </button>
  </div>
</template> 