<script setup lang="ts">
import Graphic from "./components/Graphic.vue";
import InputExcel from "./components/InputExcel.vue";
import { computed, ref } from "vue";
import { colors } from "./constants";

// Definir tipos
interface Container {
  bay: number;
  row: number;
  tier: number;
  podLetter: string; // Letra del puerto de descarga
}

interface LegendItem {
  letter: string;
  pod: string;
  color: string; // Color asignado
}

const containers = ref<Container[]>([]);
const legend = ref<LegendItem[]>([]);

const handlePrint = () => {
  window.print();
};

const handleDataProcessed = (data: { containers: any[]; legend: any[] }) => {
  // Crear un mapeo de POD a letras
  const podToLetterMap: Record<string, string> = {};
  data.legend.forEach((item, index) => {
    podToLetterMap[item.pod] = String.fromCharCode(65 + index); // A, B, C, ...
  });

  // Mapear los contenedores para agregar la propiedad podLetter
  containers.value = data.containers.map((container) => ({
    ...container,
    podLetter: podToLetterMap[container.pod], // Asignar la letra correspondiente
  }));

  // Actualizar la leyenda
  legend.value = data.legend.map((item, index) => ({
    letter: String.fromCharCode(65 + index),
    pod: item.pod,
    color: colors[index % colors.length], // Asignar color cíclicamente
  }));
};

// Crear un mapeo de colores basado en la leyenda
const podColorMap = computed(() => {
  const map = new Map<string, string>();
  legend.value.forEach((item) => {
    map.set(item.letter, item.color);
  });
  return map;
});
</script>

<template>
  <div class="min-h-screen bg-white not-print:p-4">
    <div class="print:hidden mb-2">
      <InputExcel @dataProcessed="handleDataProcessed" />

      <!-- Botón de imprimir -->
      <button
        v-if="containers.length"
        @click="handlePrint"
        class="mt-4 px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 print:hidden"
      >
        Imprimir
      </button>
    </div>
    <Graphic
      v-if="containers.length"
      :containers="containers"
      :podColorMap="podColorMap"
      :legend="legend"
    />
  </div>
</template>
