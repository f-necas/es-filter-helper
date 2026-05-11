<script setup>
import { computed } from 'vue'
const props = defineProps({
  data: { type: [Object, Array], required: true },
  path: { type: String, default: '' },
  depth: { type: Number, default: 0 }
})
const emit = defineEmits(['field-click'])

function handleClick(path, value) {
  const cleanPath = path.replace(/\.\d+/, '');
  console.log('Clicked path:', cleanPath, 'value:', value);
  if (typeof value !== 'object' || value === null) emit('field-click', { path: cleanPath, value });
}

function buildPath(parent, key, isArray) {
  const keyStr = String(key)
  if (!parent) return keyStr
  if (parent === '') return keyStr
  return parent + '.' + keyStr
}

function formatValue(value) {
  if (value === null) return 'null'
  if (typeof value === 'string') {
    return value.length > 50 ? value.substring(0, 50) + '...' : value
  }
  if (typeof value === 'boolean') return value ? 'true' : 'false'
  return String(value)
}

const hasChildren = computed(() => {
  return Object.keys(props.data).length > 0
})
</script>
<template>
  <ul class="json-tree" :style="{ paddingLeft: depth === 0 ? '0' : '1.5em' }">
    <template v-for="(value, key) in data" :key="key">
      <li class="tree-item">
        <span v-if="typeof value === 'object' && value !== null" class="tree-node">
          <span class="node-key">
            <i class="node-icon">►</i>
            <strong>{{ buildPath(path, key, Array.isArray(data)) }}</strong>
          </span>
          <JsonTree
            :data="value"
            :path="buildPath(path, key, Array.isArray(data))"
            :depth="depth + 1"
            @field-click="emit('field-click', $event)"
          />
        </span>
        <span v-else class="tree-leaf" @click="handleClick(buildPath(path, key, Array.isArray(data)), value)">
          <span class="leaf-key">{{ buildPath(path, key, Array.isArray(data)) }}</span>
          <span class="leaf-separator">:</span>
          <span class="leaf-value">{{ formatValue(value) }}</span>
        </span>
      </li>
    </template>
  </ul>
</template>

<style scoped>
.json-tree {
  list-style: none;
  margin: 0;
  padding: 0;
}

.tree-item {
  margin: 0.3em 0;
  font-size: 0.95em;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', 'Courier New', monospace;
}

.tree-node {
  display: block;
  user-select: none;
}

.node-key {
  display: flex;
  align-items: center;
  gap: 0.4em;
  color: #303133;
  cursor: pointer;
}

.node-icon {
  display: inline-block;
  width: 1em;
  text-align: center;
  color: #909399;
  font-size: 0.8em;
  font-style: normal;
  transition: transform 0.2s;
}

.node-key:hover {
  color: #409eff;
}

.node-key strong {
  color: #019161;
  font-weight: 600;
}

.tree-leaf {
  display: inline;
  cursor: pointer;
  transition: background-color 0.2s;
  padding: 0.2em 0.4em;
  border-radius: 3px;
}

.tree-leaf:hover {
  background-color: #f0f9ff;
}

.leaf-key {
  color: #019161;
  font-weight: 500;
}

.leaf-separator {
  color: #909399;
  margin: 0 0.3em;
}

.leaf-value {
  color: #d84e42;
}

.tree-leaf:hover .leaf-key {
  color: #409eff;
  text-decoration: underline;
}

.tree-leaf:hover .leaf-value {
  color: #409eff;
}

@media (max-width: 768px) {
  .tree-item {
    font-size: 0.85em;
  }

  .leaf-value {
    word-break: break-word;
  }
}
</style>
