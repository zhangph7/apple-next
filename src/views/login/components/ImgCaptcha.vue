<script lang="ts" setup>
import { ref, reactive } from 'vue'
import { getCaptcha } from '@/api/common'

const captchaData = reactive({ bg: '', fragment: '', y: 0 })
const dragX = ref(0)
const startX = ref(0)
const isDragging = ref(false)
const showModal = ref(false) // 控制弹窗显示

const onRefreshCaptcha = async () => {
  const [, res] = await getCaptcha()
  if (res) {
    Object.assign(captchaData, res.data)
    dragX.value = 0
  }
}

// 开启验证弹窗
const open = () => {
  showModal.value = true
  onRefreshCaptcha()
}

const onDragStart = (e: MouseEvent | TouchEvent) => {
  isDragging.value = true
  startX.value = e instanceof MouseEvent ? e.clientX : e.touches[0].clientX
  window.addEventListener('mousemove', onDragging)
  window.addEventListener('mouseup', onDragEnd)
}

const onDragging = (e: MouseEvent | TouchEvent) => {
  if (!isDragging.value) return
  const currentX = e instanceof MouseEvent ? e.clientX : e.touches[0].clientX
  let moveX = currentX - startX.value
  if (moveX < 0) moveX = 0
  if (moveX > 250) moveX = 250 // 300(宽) - 50(苹果宽)
  dragX.value = moveX
}

const emit = defineEmits(['success'])
const onDragEnd = () => {
  isDragging.value = false
  window.removeEventListener('mousemove', onDragging)
  window.removeEventListener('mouseup', onDragEnd)
  // 将坐标传回父组件
  emit('success', Math.round(dragX.value))
  showModal.value = false
}

defineExpose({ open, onRefreshCaptcha })
</script>

<template>
  <n-modal v-model:show="showModal" preset="card" title="请完成安全验证" style="width: 340px">
    <div class="slider-container">
      <div class="img-box">
        <img :src="captchaData.bg" class="bg" />
        <img
          :src="captchaData.fragment"
          class="apple"
          :style="{ top: captchaData.y + 'px', left: dragX + 'px' }"
        />
      </div>
      <div class="slider-bar">
        <div class="slider-track">向右滑动苹果填充缺口</div>
        <div class="slider-handle" :style="{ left: dragX + 'px' }" @mousedown="onDragStart">🍎</div>
      </div>
    </div>
  </n-modal>
</template>

<style lang="scss" scoped>
.slider-container {
  .img-box {
    position: relative;
    width: 300px;
    height: 150px;
    background: #eee;
    overflow: hidden;
    border-radius: 4px;
    .bg {
      width: 100%;
      height: 100%;
    }
    .apple {
      position: absolute;
      width: 50px;
      height: 50px;
      z-index: 10;
      filter: drop-shadow(0 0 5px rgba(0, 0, 0, 0.5));
    }
  }
  .slider-bar {
    position: relative;
    height: 40px;
    background: #f7f9fa;
    border-radius: 20px;
    margin-top: 15px;
    border: 1px solid #ddd;
    .slider-track {
      line-height: 40px;
      text-align: center;
      font-size: 12px;
      color: #999;
    }
    .slider-handle {
      position: absolute;
      top: 0;
      width: 40px;
      height: 40px;
      background: #fff;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: grab;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    }
  }
}
</style>
