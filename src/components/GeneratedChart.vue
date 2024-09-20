<template>
  <div class="generated_wrapper bg-sky-200 p-4 rounded-lg">
    <h2 class="text-2xl font-bold text-center mb-2">Generated Chart</h2>
    <BarChart :spreadsheetData="spreadsheetData" />
  </div>
</template>

<script lang="ts" setup>
import BarChart from '@/components/Charts/BarChart.vue'
import { useEventBus } from '@vueuse/core'
import { ref } from 'vue'

const bus = useEventBus<string>('spreadsheet')

function listener(event: string, data: any) {
  if (event == 'fileUploaded') {
    spreadsheetData.value = data
  }
}
bus.on(listener)

const spreadsheetData = ref<any[]>([])
</script>

<style scoped>
.generated_wrapper {
  background-color: #bae6fd;
  padding: 1rem;
  border-radius: 0.5rem;
}
</style>
