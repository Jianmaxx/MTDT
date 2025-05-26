<template>
  <header class="h-14 bg-white border-b flex items-center px-4 justify-between">
  <img src="/images/left-logo.png" alt="左logo" class="h-8 object-contain" />

  <div class="flex items-center gap-4 absolute left-1/2 -translate-x-1/2">
  <img src="/images/right-logo.png" alt="右logo" class="h-8 object-contain" />
  <h1 class="text-lg font-medium">山思数字管理平台V0.1</h1>
    </div>

    <div class="overflow-hidden w-80">
      <div class="animate-scroll flex items-center gap-6 text-sm whitespace-nowrap">
        <div class="flex items-center gap-2">
          <i class="fas fa-temperature-high text-red-500" />
          <span>温度: {{ weatherData.temperature }}℃</span>
        </div>
        <div class="flex items-center gap-2">
          <i class="fas fa-droplet text-blue-500" />
          <span>湿度: {{ weatherData.humidity }}%</span>
        </div>
        <div class="flex items-center gap-2">
          <i class="fas fa-cloud-rain text-gray-500" />
          <span>降水: {{ weatherData.precipitation }}mm</span>
        </div>
        <div class="flex items-center gap-2">
          <i class="fas fa-wind text-gray-500" />
          <span>风速: {{ weatherData.windSpeed }}m/s</span>
        </div>
      </div>
    </div>
  </header>
</template>

<script setup lang="ts">
import { reactive } from 'vue'

// 环境数据
const weatherData = reactive({
  temperature: 26,
  humidity: 65,
  precipitation: 0,
  windSpeed: 3
})
</script>
