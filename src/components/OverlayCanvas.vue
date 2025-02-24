<template>
  <div class="flex flex-col items-center h-auto bg-gray-950 p-4 text-gray-100">
    <!-- Modal de advertencia -->
    <div v-if="showWarningModal" class="fixed inset-0 flex items-center backdrop-blur-sm justify-center z-50">
      <div class="bg-gray-800 p-6 rounded-lg shadow-lg max-w-md w-full">
        <h2 class="text-xl font-semibold text-gray-100 mb-4">Advertencia</h2>
        <p class="text-gray-300 mb-6">
          Al cambiar el tamaño de la imagen SEM, las cerdas que has ordenado podrían moverse de su posición original.
          ¿Deseas continuar?
        </p>
        <div class="flex justify-end space-x-4">
          <button
            @click="acceptWarning"
            class="px-4 py-2 bg-emerald-500 text-white rounded shadow-md hover:bg-emerald-400 transition"
          >
            Aceptar
          </button>
        </div>
      </div>
    </div>

    <!-- Contenedor con la imagen SEM y las cerdas superpuestas -->
    <div class="relative w-full max-w-3xl items-center rounded-lg shadow-lg mb-4" :style="{ marginBottom: imageMarginBottom + 'px' }">
      <img
          v-if="imageUrl"
          :src="imageUrl"
          alt="Imagen SEM"
          class="max-w-full max-h-[80vh] h-auto object-contain rounded-lg origin-top"
          @load="onImageLoad"
          ref="imageElement"
          :style="{ transform: `scale(${imageScale})` }"
      />
      <!-- Imágenes SVG superpuestas -->
      <img
          v-for="(svg, index) in activeSvgs"
          :key="index"
          :src="svg.url"
          alt="Cerda seleccionada"
          class="absolute cursor-move shadow-lg rounded-lg"
          :class="{ 'border border-emerald-500': selectedSvgObj === svg }"
          :style="{ top: svg.y + 'px', left: svg.x + 'px', transform: `scale(${svg.scale})`, filter: svg.filter }"
          @mousedown="startDrag(svg, $event)"
      />
    </div>

    <!-- Controles de transformación -->
    <div v-if="imageUrl && selectedSvgObj" class="w-full max-w-3xl flex flex-col items-center space-y-4 mb-4">
      <label class="font-semibold text-gray-200">Controles de transformación:</label>
      <div class="flex space-x-2">
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

    <!-- Selector y botón para añadir SVG -->
    <div v-if="imageUrl" class="w-full max-w-3xl flex flex-col items-center space-y-4 mb-4">
      <label class="font-semibold text-gray-200">Seleccionar cerda:
        <select v-model="selectedSvg" class="p-2 border border-gray-600 rounded bg-gray-800 text-gray-100 shadow-lg">
          <option v-for="cerda in cerdas" :key="cerda" :value="cerda">
            {{ obtenerNombreArchivo(cerda) }}
          </option>
        </select>
        <button
        @click="addSvg"
        class="px-4 py-2 mt-2 bg-emerald-500 text-white rounded shadow-md hover:bg-emerald-400 transition"
      >
        Añadir
      </button>
      </label>
    </div>

    <!-- Botón para cambiar el color de los SVG -->
    <div v-if="imageUrl" class="w-full max-w-3xl flex flex-col items-center space-y-4 mb-4">
      <button
        @click="toggleSvgColor"
        class="px-4 py-2 bg-purple-500 text-white rounded shadow-md hover:bg-purple-400 transition"
      >
        Cambiar color (Negro/Blanco)
      </button>
    </div>

    <!-- Input para cargar imagen -->
    <div class="w-full max-w-3xl p-4 border rounded-lg bg-white shadow text-gray-800 mb-4 text-center">
      <input type="file" @change="onFileChange" accept="image/*" class="mb-2" />
    </div>

    <!-- Botones de zoom fuera del contenedor de la imagen -->
    <div v-if="imageUrl" class="fixed right-4 top-1/2 transform -translate-y-1/2 flex flex-col space-y-2">
      <button
        @click="handleZoomIn"
        class="px-4 py-2 bg-blue-500 text-white rounded shadow-md hover:bg-blue-400 transition"
      >
        +
      </button>
      <button
        @click="handleZoomOut"
        class="px-4 py-2 bg-red-500 text-white rounded shadow-md hover:bg-red-400 transition"
      >
        -
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";

const cerdas = ref([]);
const selectedSvg = ref("");
const activeSvgs = ref([]);
const selectedSvgObj = ref(null);
const imageElement = ref(null); // Referencia al elemento de la imagenaraeae
const imageCenter = ref({ x: 0, y: 0 }); // Centro de la imagen
const imageScale = ref(1); // Escala de la imagen SEM
const imageMarginBottom = ref(0); // Margen inferior del contenedor de la imagen
const isWhite = ref(false); // Estado para alternar entre negro y blanco
const showWarningModal = ref(false); // Estado para mostrar el modal de advertencia
const hasShownWarning = ref(false); // Estado para controlar si ya se mostró la advertencia

const imageUrl = ref(null);

const onFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    // Limpiar el estado actual antes de cargar la nueva imagen
    imageUrl.value = null; // Reiniciar la URL de la imagen
    activeSvgs.value = []; // Limpiar las cerdas superpuestas
    imageScale.value = 1; // Reiniciar la escala de la imagen
    imageMarginBottom.value = 0; // Reiniciar el margen inferior

    // Cargar la nueva imagen
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
    updateImageMargin(); // Actualizar el margen al cargar la imagen
  }
};

const addSvg = () => {
  const svgImage = new Image();
  svgImage.src = selectedSvg.value;

  svgImage.onload = () => {
    // Calculamos el centro del SVG usando sus dimensiones
    const svgCenterX = svgImage.width / 2;
    const svgCenterY = svgImage.height / 2;

    // Calculamos la posición inicial del SVG teniendo en cuenta el centro de la imagen SEM
    const x = imageCenter.value.x - svgCenterX;
    const y = imageCenter.value.y - svgCenterY;

    // Añadimos el SVG con el filtro de color actual
    activeSvgs.value.push({
      url: selectedSvg.value,
      x: x,
      y: y,
      scale: 1,
      filter: isWhite.value ? "brightness(0) invert(1)" : "none",
    });
  };
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

// Función para manejar el zoom in
const handleZoomIn = () => {
  if (!hasShownWarning.value) {
    showWarningModal.value = true;
  } else {
    zoomIn();
  }
};

// Función para manejar el zoom out
const handleZoomOut = () => {
  if (!hasShownWarning.value) {
    showWarningModal.value = true;
  } else {
    zoomOut();
  }
};

// Función para aplicar el zoom in
const zoomIn = () => {
  imageScale.value += 0.1;
  updateImageMargin(); // Actualizar el margen al hacer zoom
};

// Función para aplicar el zoom out
const zoomOut = () => {
  imageScale.value -= 0.1;
  updateImageMargin(); // Actualizar el margen al hacer zoom
};

// Función para aceptar la advertencia
const acceptWarning = () => {
  showWarningModal.value = false;
  hasShownWarning.value = true;
};

// Función para alternar el color de los SVG entre negro y blanco
const toggleSvgColor = () => {
  isWhite.value = !isWhite.value;
  activeSvgs.value.forEach(svg => {
    svg.filter = isWhite.value ? "brightness(0) invert(1)" : "none";
  });
};

// Función para calcular y actualizar el margen inferior del contenedor de la imagen
const updateImageMargin = () => {
  if (imageElement.value) {
    const imageHeight = imageElement.value.clientHeight * imageScale.value;
    const containerHeight = imageElement.value.parentElement.clientHeight;
    if (imageHeight > containerHeight) {
      imageMarginBottom.value = imageHeight - containerHeight + 20; // Añadir un pequeño espacio adicional
    } else {
      imageMarginBottom.value = 0; // Restablecer el margen si no es necesario
    }
  }
};

// Observar cambios en el zoom para actualizar el margen
watch(imageScale, updateImageMargin);
</script>
