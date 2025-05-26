<template>
  <div class="w-full h-full bg-white">
    <!-- 顶部导航栏 -->
    <Header />

    <!-- 主要内容区域 -->
    <main class="flex h-[calc(100vh-56px)] overflow-hidden">
      <!-- 左侧工具栏 -->
      <Sidebar @tool-click="handleToolClick" />

      <!-- 中间地图区域 -->
      <CesiumMap ref="mapRef" />

      <!-- 右侧监控面板 -->
      <MonitorPanel
        @mode-change="handleModeChange"
        @fullscreen-toggle="handleFullscreenToggle"
      />
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Header from './components/Header.vue'
import Sidebar from './components/Sidebar.vue'
import CesiumMap from './components/CesiumMap.vue'
import MonitorPanel from './components/MonitorPanel.vue'

const mapRef = ref()

// 处理工具栏点击事件
const handleToolClick = (toolId: string) => {
  console.log('工具被点击:', toolId)
  // 这里可以根据不同的工具ID执行相应的操作
  switch (toolId) {
    case 'edit':
      console.log('进入编辑模式')
      break
    case 'layers':
      console.log('打开图层管理')
      break
    case 'statistics':
      console.log('显示统计信息')
      break
    case 'drone':
      console.log('打开无人机控制')
      break
    case 'ai':
      console.log('启动AI模块')
      break
  }
}

// 处理工作模式切换
const handleModeChange = (modeId: string) => {
  console.log('工作模式切换到:', modeId)
  // 这里可以根据不同的模式执行相应的操作
}

// 处理全屏切换
const handleFullscreenToggle = (type: string) => {
  console.log('切换全屏:', type)
  // 这里可以实现全屏逻辑
}
</script>
