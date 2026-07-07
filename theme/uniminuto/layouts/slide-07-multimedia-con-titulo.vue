<script setup>
import { ref, onMounted, onBeforeUnmount, nextTick } from 'vue'
import { assetUrl } from '../utils/asset-url.js'
import AutoFitText from '../components/AutoFitText.vue'

const mediaRef = ref(null)
let observer = null

function normalizeAllow(value = '') {
  const parts = value
    .split(';')
    .map((part) => part.trim())
    .filter(Boolean)

  const required = [
    'accelerometer',
    'autoplay',
    'clipboard-write',
    'encrypted-media',
    'gyroscope',
    'picture-in-picture',
    'web-share',
    'fullscreen'
  ]

  for (const permission of required) {
    const exists = parts.some(
      (part) => part === permission || part.startsWith(`${permission} `)
    )

    if (!exists) {
      parts.push(permission)
    }
  }

  return parts.join('; ')
}

function enhanceIframe(iframe) {
  iframe.setAttribute('allowfullscreen', '')
  iframe.setAttribute('webkitallowfullscreen', '')
  iframe.setAttribute('mozallowfullscreen', '')
  iframe.setAttribute('allow', normalizeAllow(iframe.getAttribute('allow') || ''))

  const src = iframe.getAttribute('src') || ''

  if (src.includes('youtube.com') || src.includes('youtube-nocookie.com')) {
    if (!iframe.getAttribute('referrerpolicy')) {
      iframe.setAttribute('referrerpolicy', 'strict-origin-when-cross-origin')
    }

    iframe.setAttribute('title', iframe.getAttribute('title') || 'Recurso multimedia de YouTube')
  }

  iframe.style.border = '0'
}

function enhanceMediaIframes() {
  if (!mediaRef.value) return

  const iframes = mediaRef.value.querySelectorAll('iframe')
  iframes.forEach(enhanceIframe)
}

onMounted(async () => {
  await nextTick()
  enhanceMediaIframes()

  if (mediaRef.value) {
    observer = new MutationObserver(() => enhanceMediaIframes())
    observer.observe(mediaRef.value, {
      childList: true,
      subtree: true,
      attributes: true,
      attributeFilter: ['src', 'allow', 'allowfullscreen']
    })
  }
})

onBeforeUnmount(() => {
  if (observer) {
    observer.disconnect()
  }
})
</script>

<template>
  <div class="slide-layout">
    <img class="slide-bg" :src="assetUrl('/fondos/slide-05-template.png')" alt="Fondo institucional" />

    <div class="title-wrap">
      <AutoFitText tag="h1" class="title" :min="30" :max="60" line-height="1.04">
        <slot name="title">Agrega un título</slot>
      </AutoFitText>
    </div>

    <div ref="mediaRef" class="media-wrap uni-media-slot">
      <slot name="media" />
    </div>
  </div>
</template>

<style scoped>
.slide-layout {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: #ffffff;
}

.slide-bg {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 1;
}

.title-wrap {
  position: absolute;
  z-index: 2;
  left: 3.4%;
  top: 1.2%;
  width: 59.2%;
  height: 11.4%;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  padding-left: 0.75rem;
  box-sizing: border-box;
}

.title {
  margin: 0 !important;
  color: #ffffff !important;
  font-family: var(--font-title, 'Merriweather Sans', Arial, sans-serif);
  font-weight: 900;
  letter-spacing: -0.025em;
  text-wrap: balance;
}

.media-wrap {
  position: absolute;
  z-index: 2;
  left: 10%;
  top: 16.5%;
  width: 80%;
  height: 74%;
  overflow: visible;
  background: transparent;
}

.media-wrap :deep(p) {
  margin: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.media-wrap :deep(img),
.media-wrap :deep(video),
.media-wrap :deep(iframe) {
  display: block;
  width: 100%;
  height: 100%;
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  object-position: center;
  border: 0;
}

.media-wrap :deep(iframe[src*="youtube.com"]),
.media-wrap :deep(iframe[src*="youtube-nocookie.com"]) {
  background: #000000;
}

.media-wrap :deep(iframe:fullscreen) {
  width: 100vw !important;
  height: 100vh !important;
  max-width: 100vw !important;
  max-height: 100vh !important;
  object-fit: contain;
  background: #000000;
}
</style>
