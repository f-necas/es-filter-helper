<script setup>
import { ref, computed, toRaw, isProxy } from 'vue'
const props = defineProps(['field', 'value'])
const must = ref(true)
const copied = ref(false)

const reserved = /[+\=&|><!(){}\[\]^"~*?:\\/]/g
function escapeSegment(segment) {
  return String(segment).replace(reserved, '\\$&')
}

const filter = computed(() => {
  const prefix = must.value ? '+' : ''
  // Split path by dot, escape each segment, then join
  const field = props.field.split('.').map(escapeSegment).join('.')
  let value = props.value
  if (isProxy(value)) value = toRaw(value)
  if (typeof value === 'string') {
    return `${prefix}${field}:"${value}"`
  } else if (typeof value === 'number' || typeof value === 'boolean') {
    return `${prefix}${field}:${value}`
  } else {
    return `${prefix}${field}:${JSON.stringify(value)}`
  }
})

function copyToClipboard() {
  navigator.clipboard.writeText(filter.value)
  copied.value = true
  setTimeout(() => {
    copied.value = false
  }, 2000)
}
</script>
<template>
  <el-card class="filter-card" shadow="hover">
    <template #header>
      <div class="card-header">
        <span class="card-title">🔤 Filter Preview</span>
        <el-button
          link
          type="primary"
          @click="copyToClipboard"
          size="small"
        >
          {{ copied ? '✓ Copied!' : '📋 Copy' }}
        </el-button>
      </div>
    </template>

    <div class="filter-content">
      <div class="field-info">
        <p class="info-label">Field:</p>
        <el-tooltip :content="props.field" placement="top">
          <div class="field-name">{{ props.field }}</div>
        </el-tooltip>
      </div>

      <div class="value-info">
        <p class="info-label">Value:</p>
        <div class="value-display">{{ props.value }}</div>
      </div>

      <div class="filter-options">
        <el-checkbox v-model="must" label="Require match (+)" />
        <el-button
          type="text"
          size="small"
          @click="must = !must"
          style="margin-left: 1em;"
        >
          Toggle
        </el-button>
      </div>

      <div class="filter-preview">
        <p class="preview-label">Query String:</p>
        <pre class="filter-query">{{ filter }}</pre>
      </div>
    </div>
  </el-card>
</template>

<style scoped>
.filter-card {
  border-radius: 8px;
  border: 1px solid #dcdfe6;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.card-title {
  font-size: 1.05em;
  font-weight: 600;
  color: #303133;
}

.filter-content {
  display: flex;
  flex-direction: column;
  gap: 1.2em;
}

.field-info,
.value-info {
  display: flex;
  flex-direction: column;
  gap: 0.5em;
}

.info-label {
  margin: 0;
  font-size: 0.85em;
  color: #909399;
  font-weight: 500;
  text-transform: uppercase;
}

.field-name {
  background: #f5f7fa;
  padding: 0.75em;
  border-radius: 4px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', 'Courier New', monospace;
  font-size: 0.9em;
  color: #303133;
  word-break: break-all;
  border: 1px solid #dcdfe6;
}

.value-display {
  background: #f5f7fa;
  padding: 0.75em;
  border-radius: 4px;
  font-size: 0.9em;
  color: #303133;
  max-height: 80px;
  overflow-y: auto;
  word-break: break-word;
  border: 1px solid #dcdfe6;
}

.filter-options {
  padding: 0.75em;
  background: #f0f9ff;
  border-radius: 4px;
  border-left: 3px solid #409eff;
}

.filter-preview {
  display: flex;
  flex-direction: column;
  gap: 0.5em;
}

.preview-label {
  margin: 0;
  font-size: 0.85em;
  color: #909399;
  font-weight: 500;
  text-transform: uppercase;
}

.filter-query {
  margin: 0;
  background: #fef0f0;
  padding: 1em;
  border-radius: 4px;
  border-left: 3px solid #f56c6c;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', 'Courier New', monospace;
  font-size: 0.9em;
  color: #303133;
  overflow-x: auto;
  word-break: break-all;
  white-space: pre-wrap;
}

.filter-query::-webkit-scrollbar {
  height: 6px;
}

.filter-query::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.filter-query::-webkit-scrollbar-thumb {
  background: #c0c4cc;
  border-radius: 3px;
}

@media (max-width: 768px) {
  .filter-card {
    border-radius: 6px;
  }

  .filter-query {
    font-size: 0.85em;
    padding: 0.75em;
  }
}
</style>
