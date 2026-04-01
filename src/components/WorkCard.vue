<script setup lang="ts">
const props = defineProps<{
  title: string;
  year: string;
  image: string;
}>();
</script>

<template>
  <div class="work-card">
    <div class="art-wrapper">
      <img :src="image" :alt="title" class="work-image" />
      <div class="art-overlay">
        <span class="view-label">VIEW WORK</span>
      </div>
    </div>
    <div class="meta">
      <h3 class="title">{{ title }}</h3>
      <span class="year">{{ year }}</span>
    </div>
  </div>
</template>

<style scoped>
.work-card {
  flex: 0 0 clamp(250px, 45vh, 450px);
  cursor: pointer;
  display: flex;
  flex-direction: column;
}

.art-wrapper {
  position: relative;
  width: 100%;
  aspect-ratio: 4/5;
  overflow: hidden;
  background-color: #1a1a1a;
  transition: transform 0.6s cubic-bezier(0.23, 1, 0.32, 1);
  flex-shrink: 0;
}

.work-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.8s cubic-bezier(0.23, 1, 0.32, 1);
  filter: grayscale(0.5) contrast(1.1);
}

.work-card:hover .art-wrapper {
  transform: scale(0.98);
}

.work-card:hover .work-image {
  transform: scale(1.1);
  filter: grayscale(0) contrast(1);
}

.art-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
  opacity: 0;
  transition: opacity 0.4s ease;
}

.work-card:hover .art-overlay {
  opacity: 1;
}

.view-label {
  font-size: 0.7rem;
  letter-spacing: 4px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  padding: 1rem 2rem;
  backdrop-filter: blur(5px);
  color: #fff;
}

.meta {
  margin-top: 1.5rem;
  display: flex;
  justify-content: space-between;
  align-items: baseline;
}

.title {
  font-size: 1.2rem;
  font-weight: 700;
  letter-spacing: -0.5px;
  text-transform: uppercase;
}

.year {
  font-size: 0.7rem;
  opacity: 0.5;
  font-family: var(--font-serif);
  font-style: italic;
}

@media (max-width: 768px) {
  .work-card {
    flex: 0 0 300px;
  }
}
</style>
