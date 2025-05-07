<script setup lang="ts">
import Graphic from "./components/Graphic.vue";
import InputExcel from "./components/InputExcel.vue";
import { ref } from "vue";

// Definir tipos
interface Container {
  bay: string;
  row: number;
  tier: number;
  podLetter: string; // Letra del puerto de descarga
}

interface LegendItem {
  letter: string;
  pod: string;
}

const containers = ref<Container[]>([]);
const legend = ref<LegendItem[]>([]);

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
  }));
};
</script>

<template>
  <div class="min-h-screen bg-gray-100 p-4">
    <div class="max-w-4xl mx-auto bg-white shadow-md rounded-lg p-6">
      <InputExcel @dataProcessed="handleDataProcessed" />
      <div v-if="legend.length" class="mt-6">
        <h2 class="text-xl font-semibold mb-2">Leyenda</h2>
        <ul class="list-disc pl-6">
          <li v-for="item in legend" :key="item.letter">
            {{ item.letter }} | {{ item.pod }}
          </li>
        </ul>
      </div>
      <Graphic v-if="containers.length" :containers="containers" />
    </div>
  </div>
</template>
