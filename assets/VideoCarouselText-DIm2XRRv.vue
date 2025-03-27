<template>
  <section>
    <h3>{{ title }}</h3>
    <div class="panel">
      <div
        ref="carouselElement"
        :id="id"
        class="relative"
        data-twe-carousel-init
        data-twe-carousel-slide
        data-twe-ride="carousel"
        data-twe-interval="9999999"
      >
        <!-- Carousel items -->
        <div class="relative w-full overflow-hidden after:clear-both after:block after:content-['']">
          <div
            v-for="(videos, i) in videoLists"
            :key="i"
            :ref="(el: any) => carouselItems[i] = el"
            :class="{ hidden: i > 0 }"
            class="video-group relative float-left -mr-[100%] w-full transition-transform duration-[600ms] ease-in-out motion-reduce:transition-none"
            data-twe-carousel-item
            style="backface-visibility: hidden"
          >
            <div
              v-for="(video, vi) in videos"
              :key="vi"
              class="flex flex-col item-center px-2"
              :style="{ width: `${100 / videos.length - 1}%`, 'margin-right': `1%` }"
            >
              <video
                :ref="(el: any) => videoRefs[i + vi] = el"
                v-lazy
                controls
                :src="video.url"
                class="w-full rounded-lg shadow-md"
                :style="{ width: `100%`}"
              ></video>
              <div class="w-full mt-3 text-xl font-bold italic leading-snug" :style="{ width: `100%`}">
                Instruction: {{ video.description }}
              </div>
            </div>
          </div>
        </div>

        <!-- Carousel indicators -->
        <div
          v-if="videoLists.length > 1"
          class="absolute bottom-0 left-0 right-0 z-[2] mx-[15%] -mb-8 flex list-none justify-center p-0"
          data-twe-carousel-indicators
        >
          <button
            v-for="(_item, i) in videoLists"
            :key="i"
            :ref="(el: any) => carouselIndicators[i] = el"
            type="button"
            :data-twe-target="`#${id}`"
            :data-twe-slide-to="i"
            class="indicator"
            aria-current="true"
            :aria-label="`Slide ${i + 1}`"
          ></button>
        </div>

        <!-- Carousel controls -->
        <button
          v-if="videoLists.length > 1"
          class="indicator-btn indicator-left-btn"
          type="button"
          :data-twe-target="`#${id}`"
          data-twe-slide="prev"
        >
          <span class="inline-block h-8 w-8">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
              stroke="currentColor" class="h-6 w-6">
              <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
            </svg>
          </span>
          <span class="sr-only">Previous</span>
        </button>

        <button
          v-if="videoLists.length > 1"
          class="indicator-btn indicator-right-btn"
          type="button"
          :data-twe-target="`#${id}`"
          data-twe-slide="next"
        >
          <span class="inline-block h-8 w-8">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
              stroke="currentColor" class="h-6 w-6">
              <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
            </svg>
          </span>
          <span class="sr-only">Next</span>
        </button>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
interface VideoItem {
  url: string
  description: string
}

interface Props {
  id?: string
  title?: string
  items?: VideoItem[]
  count?: number
}

const { props } = defineProps<{ props: Props }>()
const title = props.title || ''
const count = props.count || 1
const id = props.id || title.replaceAll(" ", "")

const items = (props.items || []).map(item => ({
  url: item.url.startsWith("assets") ? new URL(`../${item.url}`, import.meta.url).href : item.url,
  description: item.description || ""
}))

const videoLists: VideoItem[][] = []
for (let i = 0; i < items.length; i += count) {
  videoLists.push(items.slice(i, i + count))
}

import { ref, onMounted } from 'vue';
import { initTWE, Carousel } from 'tw-elements';
import { store } from '@/store';

const carouselElement = ref<HTMLElement>();
const videoRefs = ref<HTMLVideoElement[]>([]);
const carouselItems = ref<HTMLElement[]>([]);
const carouselIndicators = ref<HTMLElement[]>([]);

onMounted(async () => {
  carouselItems.value[0]?.setAttribute("data-twe-carousel-active", "")
  carouselIndicators.value[0]?.setAttribute("data-twe-carousel-active", "")
  do {
    await new Promise(resolve => setTimeout(resolve, 100))
  } while (store.tweInitializing["Carousel"])
  store.setInitializing("Carousel", true)
  initTWE({ Carousel }, { allowReinits: true, checkOtherImports: true });
  store.setInitializing("Carousel", false)

  carouselElement.value?.addEventListener('slide.twe.carousel', (v: any) => {
    const from = v.from;
    const to = v.to;
    videoRefs.value[2 * from]?.pause();
    videoRefs.value[2 * from + 1]?.pause();
  })
});
</script>


<style scoped lang="scss">
section {
    @apply w-full py-10 md:px-16 px-6;
    @apply flex flex-col justify-center items-center;
}

.panel {
    max-width: 960px;
    @apply w-full mt-2;

    &>* {
        @apply w-full mb-8;
    }

    :last-child {
        @apply mb-0;
    }
}

.video-group {
    video {
        width: 49%;
        @apply rounded-lg;
    }

    @media (max-width: 768px) {
        video {
            width: 100% !important;
        }

        div {
            width: 0;
        }
    }

    div {
        width: 1%;
    }

    * {
        @apply inline-block;
    }
}

.t2i-caption {
    @apply font-light italic md:px-20 text-center leading-snug;
}

.item-content {
    @apply flex flex-col-reverse md:flex-row md:justify-between md:items-center;

    video {
        @apply w-full md:w-1/2;
    }

    div {
        @apply md:w-1/2 md:pl-2 md:pr-20 md:mt-0;
        @apply w-full mx-2 mt-2;
        @apply font-thin leading-tight;
        text-align: justify;
    }
}
</style>
