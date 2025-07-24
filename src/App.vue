<script setup>
import { ref } from 'vue'
import InputForm from './components/InputForm.vue'
import JsonTree from './components/JsonTree.vue'
import FilterPreview from './components/FilterPreview.vue'

const jsonData = ref(null)
const selectedField = ref(null)
const selectedValue = ref(null)

async function handleUrl(url) {
  const id = url.split('/dataset/')[1]
  const response = await fetch(new URL(url).origin + '/geonetwork/srv/api/search/records/_search?bucket=bucket&relatedType=fcats', {
    method: 'POST',
    headers: { 'Accept': 'application/json', 'Content-Type': 'application/json' },
    body: JSON.stringify({ query: { ids: { values: [id] } } })
  })
  const data = await response.json()
  // Only use hits.hits[0]._source if available
  jsonData.value = data?.hits?.hits?.[0]?._source || null
  selectedField.value = null
  selectedValue.value = null
}

function showFilter({ path, value }) {
  console.log('showFilter', path, value)
  selectedField.value = path
  selectedValue.value = value
}
</script>

<template>
  <el-container style="min-height: 100vh;">
    <el-header>
      <h2>Elasticsearch Filter Builder</h2>
    </el-header>
    <el-main style="padding-right: 440px;"> <!-- Add padding to avoid overlap with fixed sidebar -->
      <InputForm @submit="handleUrl" />
      <div v-if="jsonData" style="margin-top: 2em; display: flex; gap: 2em; align-items: flex-start;">
        <div style="flex: 2;">
          <JsonTree :data="jsonData" @field-click="showFilter" />
        </div>
      </div>
      <div v-if="selectedField" class="fixed-sidebar">
        <FilterPreview :field="selectedField" :value="selectedValue" />
      </div>
    </el-main>
  </el-container>
</template>

<style scoped>
.el-header {
  background: #409eff;
  color: white;
  text-align: center;
  padding: 1em 0;
}
.fixed-sidebar {
  position: fixed;
  top: 90px;
  right: 20px;
  width: 400px;
  max-width: 90vw;
  z-index: 100;
}
</style>
