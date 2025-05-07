<script setup lang="ts">
import Graphic from "./components/Graphic.vue";
import InputExcel from "./components/InputExcel.vue";
import { computed, ref } from "vue";
import { colors } from "./constants";
import Legend from "./components/Legend.vue";

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
  <div class="min-h-screen bg-gray-100 p-4">
    <div class="mx-auto bg-white shadow-md rounded-lg p-6">
      <div class="print:hidden">
        <InputExcel @dataProcessed="handleDataProcessed" />
      </div>
      <Legend v-if="legend.length" :legend="legend" />
      <Graphic
        v-if="containers.length"
        :containers="containers"
        :podColorMap="podColorMap"
      />
    </div>
  </div>
</template>
