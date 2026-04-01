<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue';
import gsap from 'gsap';

const props = defineProps<{
  src: string;
  alt?: string;
  show: boolean;
}>();

const emit = defineEmits(['close']);

const close = () => {
  gsap.to('.viewer-overlay', {
    opacity: 0,
    duration: 0.4,
    ease: 'power2.inOut',
    onComplete: () => emit('close')
  });
};

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape') close();
};

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
  gsap.fromTo('.viewer-img', 
    { scale: 0.8, opacity: 0 },
    { scale: 1, opacity: 1, duration: 0.6, ease: 'expo.out' }
  );
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
});
</script>

<template>
  <Teleport to="body">
    <div v-if="show" class="viewer-overlay" @click="close">
      <div class="close-hint">CLICK ANYWHERE TO CLOSE / ESC</div>
      <div class="img-container">
        <img :src="src" :alt="alt" class="viewer-img" @click.stop />
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.viewer-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.95);
  backdrop-filter: blur(20px);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: zoom-out;
}

.img-container {
  width: 90vw;
  height: 90vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.viewer-img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  box-shadow: 0 50px 100px rgba(0,0,0,0.5);
}

.close-hint {
  position: absolute;
  top: 2rem;
  left: 50%;
  transform: translateX(-50%);
  color: #fff;
  font-size: 0.7rem;
  letter-spacing: 3px;
  opacity: 0.4;
  pointer-events: none;
}

@media (max-width: 768px) {
  .img-container {
    width: 100vw;
    height: 100vh;
  }
}
</style>
