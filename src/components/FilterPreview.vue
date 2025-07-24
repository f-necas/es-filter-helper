<script setup>
import { ref, computed, toRaw, isProxy } from 'vue'
const props = defineProps(['field', 'value'])
const must = ref(true)

const reserved = /[+\-=&|><!(){}\[\]^"~*?:\\/]/g
function escapeSegment(segment) {
  return String(segment).replace(reserved, '\\$&')
}
function escapeQueryString(str) {
  return String(str).replace(reserved, '\\$&')
}

const filter = computed(() => {
  const prefix = must.value ? '+' : ''
  // Split path by dot, escape each segment, then join
  console.log(props.field)
  const field = props.field.split('.').map(escapeSegment).join('.')
  let value = props.value
  if (isProxy(value)) value = toRaw(value)
  if (typeof value === 'string') {
    return `${prefix}${field}:"${escapeQueryString(value)}"`
  } else if (typeof value === 'number' || typeof value === 'boolean') {
    return `${prefix}${field}:${value}`
  } else {
    return `${prefix}${field}:${JSON.stringify(value)}`
  }
})
</script>
<template>
  <el-card>
    <h4>Query String Filter Preview</h4>
    <el-checkbox v-model="must">Require match (+)</el-checkbox>
    <pre>{{ filter }}</pre>
  </el-card>
</template> 