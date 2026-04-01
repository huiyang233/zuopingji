<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from 'vue';
import gsap from 'gsap';
import { ScrollToPlugin } from 'gsap/ScrollToPlugin';
import ImageViewer from './components/ImageViewer.vue';

gsap.registerPlugin(ScrollToPlugin);

// ... existing interfaces ...

const showViewer = ref(false);
const viewerSrc = ref('');

const openViewer = (src: string) => {
  viewerSrc.value = src;
  showViewer.value = true;
};

interface Work {
  id: number;
  title: string;
  year: string;
  image: string;
  description: string;
  detailImages: string[];
  story: string;
  exhibitions: string[];
  specs: string;
}

const sections = ref([
  { 
    id: 1, 
    theme: 'FLORA', 
    subtitle: 'WATERCOLOR REFLECTIONS',
    works: [
      { 
        id: 101, 
        title: 'Whispering Blooms', 
        year: '2021', 
        image: '/src/assets/projectOne/main.jpeg',
        description: 'A delicate watercolor exploration of floral ephemerality.',
        story: 'Created on April 11, 2021, this watercolor work captures the fleeting grace of garden roses. The interplay of transparent washes and deep pigments evokes a sense of light filtering through heavy spring air, where every petal seems to whisper a story of rebirth.',
        exhibitions: ['Spring Awakening Group Show, 2022', 'Solo Series: Nature’s Breath, 2023'],
        specs: 'Hand-painted Watercolor on Arches Paper, 56 x 76 cm',
        detailImages: []
      }
    ]
  },
  { 
    id: 2, 
    theme: 'VOID', 
    subtitle: 'DIGITAL SYNTHESIS',
    works: [
      { 
        id: 201, 
        title: 'Dark Matter', 
        year: '2023', 
        image: 'https://images.unsplash.com/photo-1549490349-8643362247b5?auto=format&fit=crop&q=80&w=1200',
        description: 'The invisible architecture of the digital realm.',
        story: 'Dark Matter is an exploration of what lies between the pixels. It represents the data loss and the unexpected beauty found in compression artifacts and digital decay.',
        exhibitions: ['Modern Art Museum, Shanghai 2023', 'Binary Space, Berlin 2023'],
        specs: 'Algorithmic Painting, NFT Certified',
        detailImages: [
          'https://images.unsplash.com/photo-1550684848-fac1c5b4e853?auto=format&fit=crop&q=80&w=600',
          'https://images.unsplash.com/photo-1518640467707-6811f4a6ab73?auto=format&fit=crop&q=80&w=600',
          'https://images.unsplash.com/photo-1563089145-599997674d42?auto=format&fit=crop&q=80&w=600',
          'https://images.unsplash.com/photo-1579783902614-a3fb3927b6a5?auto=format&fit=crop&q=80&w=600'
        ]
      }
    ]
  },
  { 
    id: 3, 
    theme: 'LIQUID', 
    subtitle: 'ORGANIC ECHO',
    works: [
      { 
        id: 301, 
        title: 'Fluid State', 
        year: '2024', 
        image: 'https://images.unsplash.com/photo-1579783902614-a3fb3927b6a5?auto=format&fit=crop&q=80&w=1200',
        description: 'Moments of tension frozen in high-viscosity oil.',
        story: 'Fluid State simulates the tension and release of organic fluids. It is a dialogue between the rigidity of computer logic and the unpredictable flow of liquid physics.',
        exhibitions: ['Liquid Lab, Tokyo 2024', 'Solo Exhibition, London 2024'],
        specs: 'Fluid Dynamic Simulation, Canvas Print',
        detailImages: [
          'https://images.unsplash.com/photo-1563089145-599997674d42?auto=format&fit=crop&q=80&w=600',
          'https://images.unsplash.com/photo-1541701494587-cb58502866ab?auto=format&fit=crop&q=80&w=600',
          'https://images.unsplash.com/photo-1578301978693-85fa9c0320b9?auto=format&fit=crop&q=80&w=600',
          'https://images.unsplash.com/photo-1573521193826-58c7dc2e13e3?auto=format&fit=crop&q=80&w=600'
        ]
      }
    ]
  }
]);

const currentIndex = ref(0);
const isScrolling = ref(false);
const selectedWork = ref<Work | null>(null);
const cursor = ref<HTMLElement | null>(null);

const handleScroll = (e: WheelEvent) => {
  if (selectedWork.value) return; // 详情页打开时，完全交给原生滚动，不拦截
  
  e.preventDefault(); // 仅在首页画廊模式下拦截，防止回弹
  if (isScrolling.value) return;
  
  if (e.deltaY > 20 && currentIndex.value < sections.value.length) {
    scrollToSection(currentIndex.value + 1);
  } else if (e.deltaY < -20 && currentIndex.value > 0) {
    scrollToSection(currentIndex.value - 1);
  }
};

const scrollToSection = (index: number) => {
  if (index === currentIndex.value) return;
  isScrolling.value = true;
  
  const prevIndex = currentIndex.value;
  currentIndex.value = index;
  
  gsap.to(window, {
    scrollTo: { y: index * window.innerHeight, autoKill: false },
    duration: 1.5,
    ease: 'expo.inOut',
    onComplete: () => {
      isScrolling.value = false;
    }
  });

  if (prevIndex > 0) {
    const prevSection = document.querySelectorAll('.section')[prevIndex - 1];
    gsap.to(prevSection.querySelectorAll('.animate-item'), {
      y: -50,
      opacity: 0,
      duration: 0.8,
      ease: 'power2.in'
    });
  } else {
    gsap.to('.hero-word', { y: -100, opacity: 0, duration: 0.8, stagger: 0.1 });
  }

  if (index > 0) {
    const currSection = document.querySelectorAll('.section')[index - 1];
    gsap.fromTo(currSection.querySelector('.theme-title'), 
      { y: 150, opacity: 0, skewY: 10 }, 
      { y: 0, opacity: 1, skewY: 0, duration: 1.5, delay: 0.4, ease: 'expo.out' }
    );
    gsap.fromTo(currSection.querySelector('.subtitle'), 
      { x: -50, opacity: 0 }, 
      { x: 0, opacity: 1, duration: 1, delay: 0.6, ease: 'power3.out' }
    );
    gsap.fromTo(currSection.querySelector('.work-preview'), 
      { y: 100, opacity: 0, scale: 0.9 }, 
      { y: 0, opacity: 1, scale: 1, duration: 1.5, delay: 0.5, ease: 'expo.out' }
    );
  } else {
    gsap.to('.hero-word', { y: 0, opacity: 1, duration: 1.5, stagger: 0.1, ease: 'expo.out' });
  }
};

const openDetail = (work: Work) => {
  selectedWork.value = work;
  nextTick(() => {
    gsap.from('.detail-left', { xPercent: -100, duration: 1, ease: 'power4.inOut' });
    gsap.from('.detail-right', { xPercent: 100, duration: 1, ease: 'power4.inOut' });
    gsap.from('.detail-info-item', { y: 50, opacity: 0, stagger: 0.1, delay: 0.6 });
  });
};

const closeDetail = () => {
  gsap.to('.detail-overlay', { 
    opacity: 0, 
    duration: 0.5, 
    onComplete: () => selectedWork.value = null 
  });
};

onMounted(() => {
  window.addEventListener('wheel', handleScroll, { passive: false });
  window.addEventListener('mousemove', (e) => {
    if (cursor.value) {
      gsap.to(cursor.value, { x: e.clientX, y: e.clientY, duration: 0.2 });
    }
  });

  gsap.fromTo('.hero-word', 
    { y: 200, opacity: 0 },
    { y: 0, opacity: 1, duration: 1.5, stagger: 0.2, ease: 'power4.out' }
  );
});

onUnmounted(() => {
  window.removeEventListener('wheel', handleScroll);
});
</script>

<template>
  <div ref="cursor" class="cursor"></div>
  
  <header class="header">
    <div class="logo" @click="scrollToSection(0)">
      FANG<span class="red-dot"></span>
    </div>
    <div class="nav-dots">
      <div 
        v-for="(_, i) in sections.length + 1" 
        :key="i" 
        class="dot" 
        :class="{ active: currentIndex === i }"
        @click="scrollToSection(i)"
      ></div>
    </div>
  </header>

  <div class="page-container">
    <section class="page hero-page">
      <div class="hero-content">
        <h1 class="hero-main">
          <span class="hero-word">THE</span>
          <span class="hero-word">BEYOND</span>
          <span class="hero-word">COLLECTION</span>
        </h1>
        <div class="hero-footer">
          <p>SCROLL TO ENTER THE VOID</p>
        </div>
      </div>
    </section>

    <section v-for="section in sections" :key="section.id" class="page section">
      <div class="section-content">
        <div class="text-side">
          <span class="subtitle animate-item">{{ section.subtitle }}</span>
          <h2 class="theme-title animate-item">{{ section.theme }}</h2>
        </div>
        <div class="work-preview animate-item" @click="openDetail(section.works[0])">
          <div class="img-wrapper">
            <img :src="section.works[0].image" :alt="section.theme" />
            <div class="hover-text">EXPLORE WORK</div>
          </div>
          <div class="work-meta">
            <h3>{{ section.works[0].title }}</h3>
            <span>{{ section.works[0].year }}</span>
          </div>
        </div>
      </div>
    </section>
  </div>

    <Transition name="slide">
      <div v-if="selectedWork" class="detail-overlay">
        <div class="detail-left" @click="openViewer(selectedWork.image)">
          <img :src="selectedWork.image" class="main-detail-img" />
          <div class="close-trigger" @click.stop="closeDetail">CLOSE</div>
          <div class="zoom-hint">CLICK TO ZOOM</div>
        </div>
        <div class="detail-right">
          <div class="detail-info-item">
            <span class="label">ARTWORK</span>
            <h2 class="detail-title">{{ selectedWork.title }}</h2>
            <p class="detail-year">{{ selectedWork.year }}</p>
          </div>

          <div class="detail-info-item">
            <span class="label">THE STORY</span>
            <p class="desc">{{ selectedWork.story }}</p>
          </div>

          <div class="detail-info-item">
            <span class="label">TECHNICAL SPECS</span>
            <p class="specs-text">{{ selectedWork.specs }}</p>
          </div>

          <div class="detail-info-item">
            <span class="label">EXHIBITIONS</span>
            <ul class="exhibition-list">
              <li v-for="exhibit in selectedWork.exhibitions" :key="exhibit">{{ exhibit }}</li>
            </ul>
          </div>

          <div class="detail-info-item" v-if="selectedWork.detailImages.length > 0">
            <span class="label">PROCESS & DETAILS</span>
            <div class="detail-grid">
              <div 
                v-for="(img, index) in selectedWork.detailImages" 
                :key="img" 
                class="grid-item" 
                :class="'item-' + index"
                @click="openViewer(img)"
              >
                <img :src="img" />
              </div>
            </div>
          </div>
          
          <div class="detail-footer">
            <p>© 2024 LI HEFANG. ALL RIGHTS RESERVED.</p>
          </div>
        </div>
      </div>
    </Transition>

    <ImageViewer 
      v-if="showViewer" 
      :src="viewerSrc" 
      :show="showViewer" 
      @close="showViewer = false" 
    />
</template>

<style scoped>
.page-container {
  height: 100vh;
  width: 100vw;
}

.page {
  height: 100vh;
  width: 100vw;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

.header {
  position: fixed;
  top: 0;
  width: 100%;
  padding: 3rem;
  display: flex;
  justify-content: space-between;
  z-index: 1000;
  mix-blend-mode: difference;
}

.logo {
  font-weight: 900;
  font-size: 1.8rem;
  letter-spacing: 2px;
  cursor: pointer;
  display: flex;
  align-items: baseline;
}

.red-dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  background-color: #ff3e00;
  margin-left: 6px;
}

.nav-dots {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.dot {
  width: 8px;
  height: 8px;
  border: 1px solid #fff;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s ease;
}

.dot.active {
  background: #fff;
  transform: scale(1.5);
}

.hero-word, .theme-title, .subtitle, .work-preview {
  opacity: 0;
}

.hero-page {
  background: #000;
}

.hero-main {
  font-size: clamp(4rem, 12vw, 12rem);
  font-weight: 900;
  line-height: 0.8;
  text-align: center;
}

.hero-word {
  display: block;
}

.hero-footer {
  position: absolute;
  bottom: 5rem;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
  font-size: 0.7rem;
  letter-spacing: 5px;
  opacity: 0.5;
}

.section {
  background: #0a0a0a;
}

.section-content {
  display: flex;
  align-items: center;
  gap: 4vw;
  width: 90%;
  max-width: 1600px;
  justify-content: flex-start;
}

.text-side {
  flex: 0 0 25%;
  max-width: 400px;
}

.work-preview {
  flex: 0 0 55%;
  cursor: pointer;
  max-width: 800px;
}

.theme-title {
  font-size: clamp(4rem, 8vw, 10rem);
  font-weight: 900;
  letter-spacing: -4px;
  line-height: 0.9;
  color: transparent;
  -webkit-text-stroke: 1px rgba(255,255,255,0.2);
  white-space: nowrap;
}


.subtitle {
  font-size: 0.8rem;
  letter-spacing: 8px;
  color: var(--color-accent);
  margin-bottom: 1rem;
  display: block;
}

.work-preview {
  flex: 0 0 60%;
  cursor: pointer;
  max-width: 800px;
}

.img-wrapper {
  position: relative;
  aspect-ratio: 16/9;
  overflow: hidden;
}

.img-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 1.5s cubic-bezier(0.19, 1, 0.22, 1);
}

.img-wrapper:hover img {
  transform: scale(1.1);
}

.hover-text {
  position: absolute;
  inset: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0,0,0,0.5);
  opacity: 0;
  transition: opacity 0.5s ease;
  font-size: 0.8rem;
  letter-spacing: 3px;
}

.work-preview:hover .hover-text {
  opacity: 1;
}

.work-meta {
  margin-top: 2rem;
  display: flex;
  justify-content: space-between;
}

.detail-overlay {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: flex;
}

.detail-left {
  flex: 1.2;
  background: #000;
  position: relative;
  overflow: hidden;
  cursor: zoom-in;
}

.zoom-hint {
  position: absolute;
  bottom: 3rem;
  left: 3rem;
  font-size: 0.6rem;
  letter-spacing: 3px;
  opacity: 0.3;
  color: #fff;
  transition: opacity 0.3s ease;
}

.detail-left:hover .zoom-hint {
  opacity: 1;
}

.main-detail-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.close-trigger {
  position: absolute;
  top: 3rem;
  left: 3rem;
  font-size: 0.7rem;
  letter-spacing: 4px;
  cursor: pointer;
  z-index: 10;
  padding: 1rem 2rem;
  border: 1px solid rgba(255,255,255,0.2);
  backdrop-filter: blur(10px);
}

.detail-right {
  flex: 1;
  background: #fff;
  color: #000;
  padding: 8vw;
  display: flex;
  flex-direction: column;
  gap: 6rem;
  overflow-y: auto;
  height: 100vh;
  scrollbar-width: thin;
  scrollbar-color: #eee transparent;
}

.detail-right::-webkit-scrollbar {
  width: 4px;
}

.detail-right::-webkit-scrollbar-thumb {
  background: #000;
}

.detail-year {
  font-family: var(--font-serif);
  font-style: italic;
  font-size: 1.2rem;
  margin-top: 1rem;
  opacity: 0.4;
}

.specs-text {
  font-family: monospace;
  font-size: 0.9rem;
  opacity: 0.6;
}

.exhibition-list {
  list-style: none;
  padding: 0;
}

.exhibition-list li {
  font-size: 1.1rem;
  padding: 0.8rem 0;
  border-bottom: 1px solid #eee;
}

.detail-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
}

.grid-item img {
  width: 100%;
  aspect-ratio: 1;
  object-fit: cover;
  transition: transform 0.6s ease;
}

.grid-item:hover img {
  transform: scale(1.05);
}

.detail-footer {
  margin-top: 4rem;
  padding-top: 4rem;
  border-top: 1px solid #eee;
  font-size: 0.7rem;
  letter-spacing: 2px;
  opacity: 0.4;
  text-align: center;
}

.slide-enter-active, .slide-leave-active {
  transition: transform 1s cubic-bezier(0.85, 0, 0.15, 1);
}

.slide-enter-from { transform: translateY(100%); }
.slide-leave-to { transform: translateY(-100%); }

@media (max-width: 1024px) {
  .section-content {
    flex-direction: column;
    text-align: center;
  }
  .detail-overlay {
    flex-direction: column;
  }
}
</style>
