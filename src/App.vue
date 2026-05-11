<script setup>
import { ref } from 'vue'
import InputForm from './components/InputForm.vue'
import JsonTree from './components/JsonTree.vue'
import FilterPreview from './components/FilterPreview.vue'

const jsonData = ref(null)
const selectedField = ref(null)
const selectedValue = ref(null)
const loading = ref(false)
const error = ref(null)

/**
 * Extracts the metadata ID from different URL formats:
 * - https://domain/datahub/dataset/{id}
 * - https://domain/geonetwork/srv/.../catalog.search#/metadata/{id}
 * - https://domain/geonetwork/srv/api/records/{id}/...
 */
function extractIdFromUrl(url) {
  // Format 1: /datahub/dataset/{id}
  const datasetMatch = url.match(/\/datahub\/dataset\/([^\/?#]+)/)
  if (datasetMatch) return datasetMatch[1]

  // Format 2: /#/metadata/{id}
  const metadataMatch = url.match(/#\/metadata\/([^\/?#]+)/)
  if (metadataMatch) return metadataMatch[1]

  // Format 3: /geonetwork/srv/api/records/{id}/
  const recordsMatch = url.match(/\/geonetwork\/srv\/api\/records\/([^\/?#]+)/)
  if (recordsMatch) return recordsMatch[1]

  return null
}

async function handleUrl(url) {
  loading.value = true
  error.value = null

  try {
    const id = extractIdFromUrl(url)
    if (!id) {
      throw new Error('Invalid URL format. Please use a valid datahub or GeoNetwork URL.')
    }

    const urlObj = new URL(url)
    const origin = urlObj.origin

    const response = await fetch(origin + '/geonetwork/srv/api/search/records/_search?bucket=bucket&relatedType=fcats', {
      method: 'POST',
      headers: { 'Accept': 'application/json', 'Content-Type': 'application/json' },
      body: JSON.stringify({ query: { ids: { values: [id] } } })
    })

    if (!response.ok) {
      throw new Error(`Failed to fetch metadata (HTTP ${response.status})`)
    }

    const data = await response.json()
    // Only use hits.hits[0]._source if available
    jsonData.value = data?.hits?.hits?.[0]?._source || null
    if (!jsonData.value) {
      error.value = 'No metadata found for this record.'
    }
    selectedField.value = null
    selectedValue.value = null
  } catch (err) {
    error.value = err.message
    jsonData.value = null
  } finally {
    loading.value = false
  }
}

function showFilter({ path, value }) {
  console.log('showFilter', path, value)
  selectedField.value = path
  selectedValue.value = value
}

function clearData() {
  jsonData.value = null
  selectedField.value = null
  selectedValue.value = null
  error.value = null
}
</script>

<template>
  <el-container style="min-height: 100vh;" class="app-container">
    <el-header class="app-header">
      <div class="header-content">
        <div class="header-title">
          <h1>🔍 Metadata Explorer</h1>
        </div>
      </div>
    </el-header>
    <el-main class="app-main" :style="{ paddingRight: selectedField ? '440px' : '20px' }">
      <div class="content-wrapper">
        <div class="form-container">
          <InputForm @submit="handleUrl" :loading="loading" />
          <el-alert
            v-if="error"
            type="error"
            :title="error"
            :closable="true"
            @close="error = null"
            style="margin-top: 1em;"
          />
        </div>

        <div v-if="loading" style="text-align: center; margin-top: 3em;">
          <el-spinner />
          <p>Loading metadata...</p>
        </div>

        <div v-if="jsonData" style="margin-top: 2em;">
          <div class="metadata-container">
            <div class="tree-panel">
              <div class="panel-header">
                <h3>📋 Metadata Fields</h3>
                <el-button type="text" size="small" @click="clearData">Clear</el-button>
              </div>
              <div class="tree-content">
                <JsonTree :data="jsonData" @field-click="showFilter" />
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-if="selectedField" class="fixed-sidebar">
        <FilterPreview :field="selectedField" :value="selectedValue" />
      </div>
    </el-main>
  </el-container>
</template>

<style scoped>
.app-container {
  --primary-color: #409eff;
  --secondary-color: #f5f7fa;
  --border-color: #dcdfe6;
}

.app-header {
  background: linear-gradient(135deg, #409eff 0%, #66b1ff 100%);
  color: white;
  padding: 2em !important;
  display: flex;
  align-items: center;
  justify-content: center;
}

.header-content {
  max-width: 1400px;
  margin: 0 auto;
}

.header-title h1 {
  margin: 0;
  font-size: 2em;
  font-weight: 600;
}

.header-subtitle {
  margin: 0.5em 0 0 0;
  opacity: 0.9;
  font-size: 0.95em;
}

.app-main {
  background: #f5f7fa;
  padding: 2em 20px !important;
  transition: padding-right 0.3s ease;
}

.content-wrapper {
  max-width: 1400px;
  margin: 0 auto;
}

.form-container {
  background: white;
  padding: 1.5em;
  border-radius: 8px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.08);
}

.metadata-container {
  display: flex;
  gap: 2em;
  align-items: flex-start;
}

.tree-panel {
  flex: 1;
  background: white;
  border-radius: 8px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.08);
  overflow: hidden;
}

.panel-header {
  background: #f5f7fa;
  padding: 1.2em 1.5em;
  border-bottom: 1px solid var(--border-color);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.panel-header h3 {
  margin: 0;
  font-size: 1.1em;
  color: #303133;
}

.tree-content {
  padding: 1.5em;
  max-height: 70vh;
  overflow-y: auto;
}

.tree-content::-webkit-scrollbar {
  width: 8px;
}

.tree-content::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.tree-content::-webkit-scrollbar-thumb {
  background: #c0c4cc;
  border-radius: 4px;
}

.tree-content::-webkit-scrollbar-thumb:hover {
  background: #909399;
}

.fixed-sidebar {
  position: fixed;
  top: 120px;
  right: 20px;
  width: 400px;
  max-width: 90vw;
  z-index: 100;
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@media (max-width: 1200px) {
  .fixed-sidebar {
    width: 350px;
  }
}

@media (max-width: 768px) {
  .fixed-sidebar {
    width: calc(100vw - 40px);
    top: auto;
    bottom: 20px;
  }

  .header-title h1 {
    font-size: 1.5em;
  }

  .header-subtitle {
    display: none;
  }
}
</style>
