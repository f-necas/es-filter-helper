<script setup>
import { ref } from 'vue'
const emit = defineEmits(['submit'])
const url = ref('')
function onSubmit() {
  if (url.value.trim()) emit('submit', url.value.trim())
}
function submitTest() {
  //set value to https://demo.georchestra.org/geonetwork/duckhub/datahub/dataset/FR-130006711-002-DG-2022_Veterinaires-et-ateliers-de-volailles and emit submit
  url.value = 'https://demo.georchestra.org/geonetwork/duckhub/datahub/dataset/FR-130006711-002-DG-2022_Veterinaires-et-ateliers-de-volailles'
  emit('submit', url.value.trim())
}
</script>
<template>
  <el-form @submit.prevent="onSubmit" :inline="true">
    <el-form-item label="Datahub Dataset URL">
      <el-input v-model="url" placeholder="Paste dataset URL here" style="width: 400px" />
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="onSubmit">Load</el-button>
    </el-form-item>
    <el-form-item>
      <el-button type="secondary" @click="submitTest">LoadTest</el-button>
    </el-form-item>
  </el-form>
</template> 