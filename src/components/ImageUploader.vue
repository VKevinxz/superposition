<template>
    <div class="p-4 border rounded-lg bg-white shadow">
      <input type="file" @change="onFileChange" accept="image/*" class="mb-2" />
      <div v-if="imageUrl" class="mt-4">
        <img :src="imageUrl" alt="Imagen cargada" class="max-w-full h-auto border" />
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, defineEmits } from "vue";
  
  const imageUrl = ref(null);
  const emit = defineEmits(["image-uploaded"]); // Definir el evento
  
  const onFileChange = (event) => {
    const file = event.target.files[0];
    if (file) {
      imageUrl.value = URL.createObjectURL(file);
      emit("image-uploaded", imageUrl.value); // Emitir la imagen cargada
    }
  };
  </script>
  