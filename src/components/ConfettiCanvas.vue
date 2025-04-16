<template>
  <canvas
    ref="confettiRef"
    v-show="show"
    style="
      position: fixed;
      left: 0;
      top: 0;
      width: 100vw;
      height: 100vh;
      z-index: 99999;
      pointer-events: none;
    "
  ></canvas>
</template>
<script setup lang="ts">
import { ref, watch, nextTick, onUnmounted } from 'vue'
const props = defineProps<{ show: boolean }>()
const confettiRef = ref<HTMLCanvasElement | null>(null)
let confettiInstance: unknown = null
let ConfettiGenerator: unknown

async function startConfetti() {
  if (!ConfettiGenerator) {
    ConfettiGenerator = (await import('confetti-js')).default
  }
  if (confettiInstance) {
    confettiInstance.clear()
    confettiInstance = null
  }
  if (confettiRef.value) {
    confettiInstance = new ConfettiGenerator({
      target: confettiRef.value,
      max: 120,
      size: 1.2,
      animate: true,
      props: ['circle', 'square', 'triangle', 'line'],
      clock: 25,
      width: window.innerWidth,
      height: window.innerHeight,
      start_from_edge: true,
      respawn: false,
    })
    confettiInstance.render()
  }
}

function stopConfetti() {
  if (confettiInstance) {
    confettiInstance.clear()
    confettiInstance = null
  }
}

watch(
  () => props.show,
  (val) => {
    if (val) {
      nextTick(() => startConfetti())
    } else {
      stopConfetti()
    }
  },
)

onUnmounted(stopConfetti)

defineExpose({ confettiRef })
</script>
