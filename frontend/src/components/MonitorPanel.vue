<template>
  <div class="w-[320px] bg-white border-l p-4 flex flex-col gap-4 relative">
    <!-- 无人机画面 -->
    <div class="monitor-window flex items-center justify-center text-white relative overflow-hidden" :style="{
      width: droneWidth,
      height: droneHeight,
      transform: `${droneTransform} translate(${dronePosition.x}px, ${dronePosition.y}px)`
}" ref="droneWindow" @mousedown="startDrag('drone', $event)">
      <span>无人机画面</span>
      <div class="controls absolute bottom-1 right-1 flex gap-1 z-10">
        <button class="scale-btn bg-blue-500 text-white p-1 rounded" @click.stop="scaleUp('drone')">
          <i class="fas fa-plus" />
        </button>
        <button class="reset-btn bg-gray-500 text-white p-1 rounded" @click.stop="resetWindow('drone')">
          <i class="fas fa-undo" />
        </button>
      </div>
    </div>

    <!-- 摄像头画面 -->
    <div class="monitor-window flex items-center justify-center text-white relative overflow-hidden" :style="{
      width: cameraWidth,
      height: cameraHeight,
      transform: `${cameraTransform} translate(${cameraPosition.x}px, ${cameraPosition.y}px)`
}" ref="cameraWindow" @mousedown="startDrag('camera', $event)">
      <span>摄像头画面</span>
    <div class="controls absolute bottom-1 right-1 flex gap-1 z-10">
      <button class="scale-btn bg-blue-500 text-white p-1 rounded" @click.stop="scaleUp('camera')">
        <i class="fas fa-plus" />
      </button>
      <button class="reset-btn bg-gray-500 text-white p-1 rounded" @click.stop="resetWindow('camera')">
        <i class="fas fa-undo" />
      </button>
      <div class="resize-handle absolute bottom-0 right-0 w-4 h-4 cursor-se-resize z-20"
        @mousedown.stop="startResize('camera', $event)">
      </div>
    </div>
  </div>

  <!-- 工作模式切换 -->
  <div class="flex gap-2 mt-auto">
    <button v-for="mode in workModes" :key="mode.id" :class="[
        'flex-1 py-2 whitespace-nowrap rounded-button',
        activeMode === mode.id
          ? 'bg-primary text-white'
          : 'bg-gray-100 text-gray-700'
      ]" @click="setActiveMode(mode.id)">
      {{ mode.name }}
    </button>
  </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onUnmounted } from 'vue'

interface WorkMode {
  id: string
  name: string
}

const workModes = ref<WorkMode[]>([
  { id: 'patrol', name: '巡护工作' },
  { id: 'community', name: '社区工作' },
  { id: 'monitoring', name: '监测数据' },
  { id: 'entry', name: '入区情况' }
])

const activeMode = ref('patrol')

const droneWindow = ref<HTMLElement | null>(null)
const cameraWindow = ref<HTMLElement | null>(null)

// 初始尺寸
const initialWidth = '100%'
const initialHeight = '180px'

interface Position {
  x: number
  y: number
}

// 无人机画面状态
const droneScale = ref(1)
const droneWidth = ref(initialWidth)
const droneHeight = ref(initialHeight)
const droneTransform = ref('none')
const dronePosition = ref<Position>({ x: 0, y: 0 })

// 摄像头画面状态
const cameraScale = ref(1)
const cameraWidth = ref(initialWidth)
const cameraHeight = ref(initialHeight)
const cameraTransform = ref('none')
const cameraPosition = ref<Position>({ x: 0, y: 0 })

const emit = defineEmits<{
  modeChange: [modeId: string]
}>()

const setActiveMode = (modeId: string) => {
  activeMode.value = modeId
  emit('modeChange', modeId)
}

// 开始拖拽
const startDrag = (type: 'drone' | 'camera', e: MouseEvent) => {
  e.preventDefault()
  const startX = e.clientX
  const startY = e.clientY
  const startPos = type === 'drone'
    ? { ...dronePosition.value }
    : { ...cameraPosition.value }

  const doDrag = (moveEvent: MouseEvent) => {
    const dx = moveEvent.clientX - startX
    const dy = moveEvent.clientY - startY

    if (type === 'drone') {
      dronePosition.value = {
        x: startPos.x + dx,
        y: startPos.y + dy
      }
    } else {
      cameraPosition.value = {
        x: startPos.x + dx,
        y: startPos.y + dy
      }
    }
  }

  const stopDrag = () => {
    document.removeEventListener('mousemove', doDrag)
    document.removeEventListener('mouseup', stopDrag)
  }

  document.addEventListener('mousemove', doDrag)
  document.addEventListener('mouseup', stopDrag)
}

// 放大窗口
const scaleUp = (type: 'drone' | 'camera') => {
  if (type === 'drone') {
    droneScale.value = droneScale.value === 1 ? 2 : droneScale.value === 2 ? 3 : 1
    droneTransform.value = `scale(${droneScale.value})`
  } else {
    cameraScale.value = cameraScale.value === 1 ? 2 : cameraScale.value === 2 ? 3 : 1
    cameraTransform.value = `scale(${cameraScale.value})`
  }
}

// 重置窗口
const resetWindow = (type: 'drone' | 'camera') => {
  if (type === 'drone') {
    droneWidth.value = initialWidth
    droneHeight.value = initialHeight
    droneTransform.value = 'none'
    dronePosition.value = { x: 0, y: 0 }
    droneScale.value = 1
  } else {
    cameraWidth.value = initialWidth
    cameraHeight.value = initialHeight
    cameraTransform.value = 'none'
    cameraPosition.value = { x: 0, y: 0 }
    cameraScale.value = 1
  }
}

// 开始调整大小
const startResize = (type: 'drone' | 'camera', e: MouseEvent) => {
  e.preventDefault()
  const windowElement = type === 'drone' ? droneWindow.value : cameraWindow.value
  if (!windowElement) return

  const startX = e.clientX
  const startY = e.clientY
  const startWidth = windowElement.offsetWidth
  const startHeight = windowElement.offsetHeight

  const doResize = (moveEvent: MouseEvent) => {
    const newWidth = startWidth + (moveEvent.clientX - startX)
    const newHeight = startHeight + (moveEvent.clientY - startY)

    if (type === 'drone') {
      droneWidth.value = `${newWidth}px`
      droneHeight.value = `${newHeight}px`
      droneTransform.value = 'none'
      droneScale.value = 1
    } else {
      cameraWidth.value = `${newWidth}px`
      cameraHeight.value = `${newHeight}px`
      cameraTransform.value = 'none'
      cameraScale.value = 1
    }
  }

  const stopResize = () => {
    document.removeEventListener('mousemove', doResize)
    document.removeEventListener('mouseup', stopResize)
  }

  document.addEventListener('mousemove', doResize)
  document.addEventListener('mouseup', stopResize)
}

// 清理事件监听器
onUnmounted(() => {
  document.removeEventListener('mousemove', () => { })
  document.removeEventListener('mouseup', () => { })
})
</script>

<style scoped>
.monitor-window {
  background-color: #333;
  border-radius: 4px;
  transition: all 0.2s ease;
  transform-origin: center center;
  cursor: move;
    z-index: 10;
    resize: both;
    overflow: auto;
    min-width: 100px;
    min-height: 100px;
  }.fullscreen-btn {
  background: rgba(0, 0, 0, 0.5);  border: none;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
}

.fullscreen-btn:hover {
  background: rgba(0, 0, 0, 0.7);
}

.resize-handle {
  opacity: 0;
  transition: opacity 0.2s ease;
}

.monitor-window:hover .resize-handle {
  opacity: 1;
}

.controls {
  opacity: 0;
  transition: opacity 0.2s ease;
}

.monitor-window:hover .controls {
  opacity: 1;
}

.scale-btn,
.reset-btn {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.scale-btn:hover {
  background-color: #3b82f6 !important;
}

.reset-btn:hover {
  background-color: #6b7280 !important;
}
</style>
