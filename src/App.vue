<template>
  <div class="min-h-screen flex flex-col items-center justify-center bg-gray-950 p-4">
    <ImageUploader @image-uploaded="setImage" />
    <OverlayCanvas v-if="imageUrl" :imageUrl="imageUrl" />
  </div>
</template>

<script setup>
import { ref } from "vue";
import ImageUploader from "./components/ImageUploader.vue";
import OverlayCanvas from "./components/OverlayCanvas.vue";

const imageUrl = ref(null);

const setImage = (url) => {
  imageUrl.value = url;
};
</script>
