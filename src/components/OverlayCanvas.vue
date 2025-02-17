<template>
    <div class="flex flex-col items-center h-auto bg-gray-950 p-4 text-gray-100">

      <div class="p-4 border rounded-lg bg-white shadow text-gray-800  mb-4">
        <input type="file" @change="onFileChange" accept="image/*" class="mb-2" />
        <!-- <div v-if="imageUrl" class="mt-4">
          <img :src="imageUrl" alt="Imagen cargada" class="max-w-full h-auto border" />
        </div> -->
      </div>
      
      <!-- Selector y botón para añadir SVG -->
      <div class="flex space-x-4 items-center mb-2" v-if="imageUrl">
        <label class="font-semibold text-gray-200">Seleccionar cerda:</label>
        <select v-model="selectedSvg" class="p-2 border border-gray-600 rounded bg-gray-800 text-gray-100 shadow-lg">
          <option v-for="cerda in cerdas" :key="cerda" :value="cerda">
            {{ obtenerNombreArchivo(cerda) }}
          </option>
        </select>
        <button
          @click="addSvg"
          class="px-4 py-2 bg-emerald-500 text-white rounded shadow-md hover:bg-emerald-400 transition"
        >
          Añadir
        </button>
      </div>
  
      <!-- Contenedor con la imagen SEM y las cerdas superpuestas -->
      <div class="relative w-full max-w-3xl items-center rounded-lg shadow-lg">
        <img
            v-if="imageUrl"
            :src="imageUrl"
            alt="Imagen SEM"
            class="max-w-full max-h-[80vh] h-auto object-contain rounded-lg"
            @load="onImageLoad"
            ref="imageElement"
        />
        <!-- Imágenes SVG superpuestas -->
        <img
            v-for="(svg, index) in activeSvgs"
            :key="index"
            :src="svg.url"
            alt="Cerda seleccionada"
            class="absolute cursor-move shadow-lg rounded-lg"
            :class="{ 'border border-emerald-500': selectedSvgObj === svg }"
            :style="{ top: svg.y + 'px', left: svg.x + 'px', transform: `scale(${svg.scale})` }"
            @mousedown="startDrag(svg, $event)"
        />
      </div>
  
      <!-- Controles de transformación (Renderizado solo si hay un SVG seleccionado) -->
      <div v-if="selectedSvgObj !== null" class="flex space-x-2 mt-2">
        <button @click="selectedSvgObj.scale += 0.1" class="px-4 py-2 bg-blue-500 text-white rounded shadow-md hover:bg-blue-400 transition">
          Agrandar
        </button>
        <button @click="selectedSvgObj.scale -= 0.1" class="px-4 py-2 bg-red-500 text-white rounded shadow-md hover:bg-red-400 transition">
          Reducir
        </button>
        <button @click="removeSvg" class="px-4 py-2 bg-gray-600 text-white rounded shadow-md hover:bg-gray-500 transition">
          Eliminar
        </button>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from "vue";
  
  const cerdas = ref([]);
  const selectedSvg = ref("");
  const activeSvgs = ref([]);
  const selectedSvgObj = ref(null);
  const imageElement = ref(null); // Referencia al elemento de la imagen
  const imageCenter = ref({ x: 0, y: 0 }); // Centro de la imagen
  
  const imageUrl = ref(null);
  
  const onFileChange = (event) => {
    const file = event.target.files[0];
    if (file) {
      imageUrl.value = URL.createObjectURL(file);
    }
  };

  onMounted(() => {
    cerdas.value = [
      "/cerdas/A1.svg",
      "/cerdas/A2.svg",
      "/cerdas/A3.svg",
      "/cerdas/N1.svg",
      "/cerdas/N2.1.svg",
      "/cerdas/N2.svg",
      "/cerdas/N3.svg",
      "/cerdas/N4.svg",
      "/cerdas/N5.svg",
    ];
    selectedSvg.value = cerdas.value[0];
  });
  
  const obtenerNombreArchivo = (ruta) => ruta.split("/").pop();
  
  // Cuando la imagen se carga, calculamos su centro
  const onImageLoad = () => {
    if (imageElement.value) {
      const rect = imageElement.value.getBoundingClientRect();
      imageCenter.value = {
        x: rect.width / 2,
        y: rect.height / 2,
      };
    }
  };
  
  const addSvg = () => {
    // Posicionamos el SVG en el centro de la imagen
    activeSvgs.value.push({
      url: selectedSvg.value,
      x: imageCenter.value.x,
      y: imageCenter.value.y,
      scale: 1,
    });
  };
  
  const removeSvg = () => {
    if (selectedSvgObj.value) {
      activeSvgs.value = activeSvgs.value.filter(svg => svg !== selectedSvgObj.value);
      selectedSvgObj.value = null;
    }
  };
  
  let dragging = false;
  let offsetX, offsetY;
  let currentSvg = null;
  
  const startDrag = (svg, event) => {
    selectedSvgObj.value = svg;
    dragging = true;
    currentSvg = svg;
    offsetX = event.clientX - svg.x;
    offsetY = event.clientY - svg.y;
    document.addEventListener("mousemove", onDrag);
    document.addEventListener("mouseup", stopDrag);
  };
  
  const onDrag = (event) => {
    if (dragging && currentSvg) {
      currentSvg.x = event.clientX - offsetX;
      currentSvg.y = event.clientY - offsetY;
    }
  };
  
  const stopDrag = () => {
    dragging = false;
    currentSvg = null;
    document.removeEventListener("mousemove", onDrag);
    document.removeEventListener("mouseup", stopDrag);
  };
  </script>