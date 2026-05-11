<script setup>
import { ref } from 'vue'
const props = defineProps({
  loading: { type: Boolean, default: false }
})
const emit = defineEmits(['submit'])
const url = ref('')

function onSubmit() {
  if (url.value.trim()) emit('submit', url.value.trim())
}

function submitTest() {
  url.value = 'https://demo.georchestra.org/geonetwork/duckhub/datahub/dataset/FR-130006711-002-DG-2022_Veterinaires-et-ateliers-de-volailles'
  emit('submit', url.value.trim())
}

// Fill example URLs for testing
function loadExample(index) {
  const examples = [
    'https://demo.georchestra.org/datahub/dataset/7afb3e3f-860e-4752-8f49-93d65b13440b',
    'https://demo.georchestra.org/geonetwork/srv/fre/catalog.search#/metadata/7afb3e3f-860e-4752-8f49-93d65b13440b',
    'https://demo.georchestra.org/geonetwork/srv/api/records/7afb3e3f-860e-4752-8f49-93d65b13440b/formatters/xml'
  ]
  url.value = examples[index]
  emit('submit', url.value.trim())
}
</script>
<template>
  <div class="form-wrapper">
    <div class="form-header">
      <h3>📌 Load Metadata</h3>
      <p class="form-description">Paste a metadata URL to explore its contents</p>
    </div>

    <el-form @submit.prevent="onSubmit" :inline="true" class="input-form">
      <el-form-item>
        <el-input
          v-model="url"
          placeholder="Paste dataset URL here (Datahub, GeoNetwork, or API endpoints)"
          style="width: 500px"
          :disabled="loading"
          @keyup.enter="onSubmit"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit" :loading="loading">
          {{ loading ? 'Loading...' : 'Load' }}
        </el-button>
      </el-form-item>
    </el-form>

    <div class="supported-formats">
      <p class="format-title">✓ Supported URL formats:</p>
      <div class="format-list">
        <el-tag type="success" style="margin: 0.3em; font-size: 0.9em;">
          datahub/dataset/{id}
        </el-tag>
        <el-tag type="success" style="margin: 0.3em; font-size: 0.9em;">
          catalog.search#/metadata/{id}
        </el-tag>
        <el-tag type="success" style="margin: 0.3em; font-size: 0.9em;">
          /api/records/{id}/...
        </el-tag>
      </div>
    </div>

    <div class="test-examples">
      <p class="example-title">Try these examples:</p>
      <div class="example-buttons">
        <el-button
          link
          type="primary"
          size="small"
          @click="loadExample(0)"
          :disabled="loading"
        >
          Example 1: Datahub Format
        </el-button>
        <el-button
          link
          type="primary"
          size="small"
          @click="loadExample(1)"
          :disabled="loading"
        >
          Example 2: GeoNetwork Format
        </el-button>
        <el-button
          link
          type="primary"
          size="small"
          @click="loadExample(2)"
          :disabled="loading"
        >
          Example 3: API Format
        </el-button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.form-wrapper {
  padding: 0;
}

.form-header {
  margin-bottom: 1.5em;
}

.form-header h3 {
  margin: 0 0 0.5em 0;
  font-size: 1.15em;
  color: #303133;
}

.form-description {
  margin: 0;
  color: #606266;
  font-size: 0.95em;
}

.input-form {
  display: flex;
  gap: 0.5em;
  flex-wrap: wrap;
  margin-bottom: 1.5em;
}

.supported-formats {
  background: #f0f9ff;
  border-left: 4px solid #409eff;
  padding: 1em;
  border-radius: 4px;
  margin-bottom: 1em;
}

.format-title {
  margin: 0 0 0.5em 0;
  font-size: 0.9em;
  color: #606266;
  font-weight: 500;
}

.format-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5em;
}

.test-examples {
  background: #fef0f0;
  border-left: 4px solid #f56c6c;
  padding: 1em;
  border-radius: 4px;
}

.example-title {
  margin: 0 0 0.5em 0;
  font-size: 0.9em;
  color: #606266;
  font-weight: 500;
}

.example-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5em;
}

@media (max-width: 768px) {
  .input-form {
    display: flex;
    flex-direction: column;
  }

  .input-form :deep(.el-input) {
    width: 100% !important;
  }
}
</style>
