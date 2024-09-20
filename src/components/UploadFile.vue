<template>
  <div class="upload-step_wrapper bg-sky-200 p-4 rounded-lg">
    <h2 class="text-2xl font-bold text-center mb-2">Upload your file</h2>
    <div>
      <n-upload directory-dnd accept=".xlsx" @on-error="onError" @before-upload="onSuccess">
        <n-upload-dragger>
          <div style="margin-bottom: 12px">
            <font-awesome-icon :icon="['fas', 'box-archive']" />
          </div>
          <n-text style="font-size: 16px">
            Click or drag an xlsx spreedsheed to this area to upload
          </n-text>
        </n-upload-dragger>
      </n-upload>
    </div>
  </div>
</template>

<script setup lang="ts">
import { NUploadDragger, NUpload, NIcon, NText, NP } from 'naive-ui'
import type { UploadFileInfo } from 'naive-ui'
import * as XLSX from 'xlsx'
import { ref } from 'vue'
import { useEventBus } from '@vueuse/core'
const bus = useEventBus<string>('spreadsheet')

const finalData = ref<any[]>([]) // Create a reactive variable to store the final data

function onError(err: Error) {
  window.$message.error('Upload failed. Please try again.')
}
function onSuccess({ file, event }: { file: UploadFileInfo; event: ProgressEvent }) {
  window.$message.success('Upload successful!')

  const reader = new FileReader()
  reader.onload = (e) => {
    const data = e.target?.result
    if (data) {
      const workbook = XLSX.read(data, { type: 'array' })
      const firstSheetName = workbook.SheetNames[0]
      const worksheet = workbook.Sheets[firstSheetName]
      const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 })
      finalData.value = jsonData
      bus.emit('fileUploaded', finalData.value)
    }
  }
  reader.readAsArrayBuffer(file.file as Blob)
}
</script>

<style scoped></style>
