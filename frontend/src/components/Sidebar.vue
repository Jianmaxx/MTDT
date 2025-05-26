<template>
  <div class="w-14 bg-gray-800 flex flex-col gap-2 p-2">
    <button
      v-for="tool in tools"
      :key="tool.id"
      class="tool-button group relative"
      @click="handleToolClick(tool.id)"
    >
      <i :class="tool.icon" />
      <span class="absolute left-full ml-2 px-2 py-1 bg-gray-800 text-white text-sm rounded opacity-0 group-hover:opacity-100 whitespace-nowrap">
        {{ tool.name }}
      </span>
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface Tool {
  id: string
  name: string
  icon: string
}

const tools = ref<Tool[]>([
  { id: 'edit', name: '编辑', icon: 'fas fa-edit' },
  { id: 'layers', name: '图层管理', icon: 'fas fa-layer-group' },
  { id: 'statistics', name: '统计', icon: 'fas fa-chart-line' },
  { id: 'drone', name: '无人机控制', icon: 'fas fa-helicopter' },
  { id: 'ai', name: 'AI模块', icon: 'fas fa-robot' }
])

const emit = defineEmits<{
  toolClick: [toolId: string]
}>()

const handleToolClick = (toolId: string) => {
  console.log('Tool clicked:', toolId)
  emit('toolClick', toolId)
}
</script>
