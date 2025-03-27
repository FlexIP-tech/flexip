<template>
    <section class="gallery">
        <h3>{{title}}</h3>
        <div class="image-grid">
            <div v-for="(img, index) in images" :key="index" class="image-item">
                <img :src="img.url" class="styled-image" @error="console.error('Image failed to load:', img.url)">
                <p class="image-description">{{ descriptions[index] }}</p>
            </div>
        </div>
    </section>
</template>

<script setup lang="ts">
interface ImageItem {
    url: string;
    desc?: string;
}

interface Props {
    title: string,
    images?: ImageItem[];
}

const { props } = defineProps<{ props: Props }>()

import { computed } from 'vue';

// Process images array
const images = computed(() => 
  props.images?.map(item => ({
    url: item.url.startsWith("assets") 
      ? new URL(`../${item.url}`, import.meta.url).href 
      : item.url,
    desc: item.desc
  }))
)
const title = props.title

// Handle descriptions array
const descriptions = computed(() => 
  props.images?.map(item => item.desc) || []
)
</script>

<style lang="scss" scoped>
.gallery {
    h3 {
        @apply mb-6 text-center text-3xl font-bold;
    }

    .image-grid {
        @apply w-full grid grid-cols-1 gap-8 max-w-6xl mx-auto; // Single column layout
        
        @media (min-width: 768px) {
            @apply grid-cols-1 md:grid-cols-1 lg:grid-cols-1; // Explicit single column for all sizes
        }
    }

    .image-item {
        @apply overflow-hidden rounded-lg shadow-sm;

        img.styled-image {
            @apply w-full h-full object-cover transition-transform duration-300 hover:scale-105;
            
            /* Add shadow effect */
            filter: drop-shadow(4px 4px 8px rgba(0, 0, 0, 0.1));
            
            /* Add gradient background overlay */
            &::after {
                content: '';
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                background: linear-gradient(rgba(0, 0, 0, 0.3), rgba(0, 0, 0, 0.1));
                pointer-events: none;
            }
        }

        p.image-description {
            @apply mt-2 text-center text-sm text-gray-600;
        }
    }
}
</style>