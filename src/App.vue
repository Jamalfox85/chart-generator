<script setup lang="ts">
import TheHeader from '@/components/TheHeader.vue'
import UploadFile from '@/components/UploadFile.vue'
import GeneratedChart from '@/components/GeneratedChart.vue'
</script>

<template>
  <TheHeader />
  <main class="p-8">
    <UploadFile class="mb-8" />
    <GeneratedChart />
  </main>
</template>

<style scoped></style>
