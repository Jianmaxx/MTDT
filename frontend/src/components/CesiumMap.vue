<template>
  <div class="flex-1 relative">
    <!-- 地图容器 -->
    <div ref="cesiumContainer" class="w-full h-full" />

    <!-- 加载状态指示器 -->
    <div v-if="loading" class="absolute inset-0 bg-gray-100 flex items-center justify-center">
      <div class="text-center">
        <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto mb-4"></div>
        <p class="text-gray-600">正在加载地图...</p>
      </div>
    </div>

    <!-- 错误状态指示器 -->
    <div v-if="error" class="absolute inset-0 bg-red-50 flex items-center justify-center">
      <div class="text-center p-8">
        <div class="text-red-600 text-xl mb-4">
          <i class="fas fa-exclamation-triangle"></i>
        </div>
        <h3 class="text-lg font-semibold text-red-800 mb-2">地图加载失败</h3>
        <p class="text-red-600 mb-4">{{ error }}</p>
        <button @click="retryInit" class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700">
          重试
        </button>
      </div>
    </div>

    <!-- 地图控制按钮 -->
    <div v-if="!loading && !error" class="absolute right-4 bottom-4 flex gap-2 z-10">
      <button class="tool-button" @click="zoomIn" title="放大">
        <i class="fas fa-plus" />
      </button>
      <button class="tool-button" @click="zoomOut" title="缩小">
        <i class="fas fa-minus" />
      </button>
      <button class="tool-button" @click="goToLocation" title="定位">
        <i class="fas fa-location-dot" />
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as Cesium from 'cesium'

const cesiumContainer = ref<HTMLElement>()
let viewer: Cesium.Viewer | null = null

// 状态管理
const loading = ref(true)
const error = ref('')

// Cesium Ion token
const CESIUM_TOKEN = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI0OTIyYjA2NS1kOTRlLTQ5YjgtODI0Yi0wNjNmNmQ1MzVlMzgiLCJpZCI6Mjg4MzQxLCJpYXQiOjE3NDQwMzgwNjl9._Wk3WsAUjBsz50AW1CfNYW9tsEbkJYIBLpVxZSFhZa8'

const initializeCesium = async () => {
  if (!cesiumContainer.value) {
    console.error('Cesium container not found')
    error.value = 'Cesium 容器未找到'
    loading.value = false
    return
  }

  try {
    loading.value = true
    error.value = ''
    console.log('Initializing Cesium...')
    
    // 设置Cesium Ion token
    Cesium.Ion.defaultAccessToken = CESIUM_TOKEN
    console.log('Cesium Ion token set')

    // 先创建基础viewer，然后异步加载地形
    viewer = new Cesium.Viewer(cesiumContainer.value, {
      animation: false,
      baseLayerPicker: false,
      fullscreenButton: false,
      vrButton: false,
      geocoder: false,
      homeButton: false,
      infoBox: false,
      sceneModePicker: false,
      selectionIndicator: false,
      timeline: false,
      navigationHelpButton: false,
      creditContainer: document.createElement('div')
    })

    console.log('Cesium viewer created')

    // 异步加载地形
    try {
      const terrainProvider = await Cesium.createWorldTerrainAsync()
      viewer.terrainProvider = terrainProvider
      console.log('Terrain provider loaded')
    } catch (terrainError) {
      console.warn('Failed to load terrain provider, using default terrain:', terrainError)
    }

    // 启用光照
    viewer.scene.globe.enableLighting = true

    // 设置初始视角到中国
    viewer.camera.setView({
      destination: Cesium.Cartesian3.fromDegrees(104.7196, 32.4842, 20000),
    })

    loading.value = false
    console.log('Cesium viewer initialized successfully')
  } catch (err: any) {
    console.error('Failed to initialize Cesium viewer:', err)
    error.value = err.message || '地图初始化失败'
    loading.value = false
  }
}

const retryInit = () => {
  if (viewer) {
    viewer.destroy()
    viewer = null
  }
  initializeCesium()
}

onMounted(() => {
  initializeCesium()
})

onBeforeUnmount(() => {
  if (viewer) {
    viewer.destroy()
    viewer = null
  }
})

// 地图控制方法
const zoomIn = () => {
  if (viewer) {
    const camera = viewer.camera
    const currentHeight = camera.positionCartographic.height
    camera.zoomIn(currentHeight * 0.2)
  }
}

const zoomOut = () => {
  if (viewer) {
    const camera = viewer.camera
    const currentHeight = camera.positionCartographic.height
    camera.zoomOut(currentHeight * 0.2)
  }
}

const goToLocation = () => {
  if (!viewer) return;

  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { longitude, latitude } = position.coords;
        viewer!.camera.flyTo({
          destination: Cesium.Cartesian3.fromDegrees(longitude, latitude, 2000),
          duration: 2.0
        });
      },
      (error) => {
        console.error('获取位置失败:', error);
        // 失败时回退到默认位置
        viewer!.camera.flyTo({
          destination: Cesium.Cartesian3.fromDegrees(116.397, 39.908, 10000000),
          duration: 2.0
        });
      }
    );
  } else {
    // 浏览器不支持定位时使用默认位置
    viewer.camera.flyTo({
      destination: Cesium.Cartesian3.fromDegrees(116.397, 39.908, 10000000),
      duration: 2.0
    });
  }
}

// 暴露viewer实例供父组件使用
defineExpose({
  viewer: () => viewer
})
</script>