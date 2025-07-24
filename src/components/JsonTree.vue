<script setup>
import { computed } from 'vue'
const props = defineProps({
  data: { type: [Object, Array], required: true },
  path: { type: String, default: '' }
})
const emit = defineEmits(['field-click'])
function handleClick(path, value) {
  const cleanPath = path.replace(/\.\d+/, '');
  // eslint-disable-next-line no-console
  console.log('Clicked path:', cleanPath, 'value:', value);
  if (typeof value !== 'object' || value === null) emit('field-click', { path: cleanPath, value });
}
function buildPath(parent, key, isArray) {
  // Always use string for key
  const keyStr = String(key)
  if (!parent) return keyStr
  // Avoid accidental undefined or double dots
  if (parent === '') return keyStr
  return parent + '.' + keyStr
}
</script>
<template>
  <ul style="list-style: none; padding-left: 1em;">
    <template v-for="(value, key) in data" :key="key">
      <li>
        <span v-if="typeof value === 'object' && value !== null">
          <strong>{{ buildPath(path, key, Array.isArray(data)) }}</strong>:
          <JsonTree :data="value" :path="buildPath(path, key, Array.isArray(data))" @field-click="emit('field-click', $event)" />
        </span>
        <span v-else @click="handleClick(buildPath(path, key, Array.isArray(data)), value)" style="cursor:pointer; color:#409eff;">
          {{ buildPath(path, key, Array.isArray(data)) }}: {{ value }}
        </span>
      </li>
    </template>
  </ul>
</template>