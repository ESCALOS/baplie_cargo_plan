<script setup lang="ts">
interface Container {
  bay: number; // Número de la bahía a la que pertenece el contenedor
  row: number;
  tier: number;
  podLetter: string; // Letra del puerto de descarga
}

const props = defineProps<{
  bay: string; // Título de la bahía (puede ser "BAHÍA 001/002")
  maxRow: number;
  containers: Container[];
  podColorMap: Map<string, string>;
}>();

// Generar las columnas (ROW) con 00 en el centro, pares a la izquierda e impares a la derecha
const rows = Array.from({ length: props.maxRow + 1 }, (_, i) => i)
  .filter((row) => row !== 0) // Excluir el 0 temporalmente
  .reduce<number[]>(
    (acc, row) => {
      if (row % 2 === 0) {
        acc.unshift(row); // Pares a la izquierda
      } else {
        acc.push(row); // Impares a la derecha
      }
      return acc;
    },
    [0]
  ); // Incluir el 00 en el centro

// Generar las filas (TIER)
const tiers = [
  ...Array.from({ length: 13 }, (_, i) => 96 - i * 2), // Parte superior (96 al 72)
  ...Array.from({ length: 11 }, (_, i) => 22 - i * 2), // Parte inferior (22 al 02)
];

// Formatear los valores de ROW y TIER para que tengan dos dígitos
const formatTwoDigits = (value: number) => value.toString().padStart(2, "0");

// Saber si es una bahia 4N+3
const isUpperBay = (bay: string) => {
  const baseBay = parseInt(bay.split("/")[0]);
  return baseBay % 4 === 3; // Bahías 4N+3
};
</script>

<template>
  <div class="mb-8">
    <h2 class="font-bold mb-2 text-center">{{ bay }}</h2>
    <div class="overflow-auto">
      <table class="table-auto border-collapse border border-gray-300">
        <thead>
          <tr>
            <th class="border border-gray-300 p-1 bg-gray-100 text-xs"></th>
            <th
              v-for="row in rows"
              :key="row"
              class="border border-gray-300 p-1 bg-gray-100 text-center text-xs"
            >
              {{ formatTwoDigits(row) }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="tier in tiers" :key="tier">
            <td
              class="border border-gray-300 p-1 bg-gray-100 text-center text-xs"
            >
              {{ formatTwoDigits(tier) }}
            </td>
            <td
              v-for="row in rows"
              :key="row"
              class="border border-gray-300 p-1 text-center text-xs"
              :style="{
                backgroundColor: podColorMap.get(
                  containers.find(
                    (container) =>
                      container.row === row && container.tier === tier
                  )?.podLetter || ''
                ),
              }"
            >
              <!-- Mostrar la letra del puerto de descarga -->
              <span
                v-if="
                  containers.some(
                    (container) =>
                      container.row === row && container.tier === tier
                  )
                "
              >
                {{
                  (() => {
                    const container = containers.find(
                      (c) => c.row === row && c.tier === tier
                    );
                    if (!container) return "";
                    if (
                      isUpperBay(props.bay) &&
                      (container.bay - 2) % 4 === 0
                    ) {
                      return ""; // Es un contenedor de 4N+2 dentro de una bahía 4N+3, no mostrar letra
                    }
                    return container.podLetter;
                  })()
                }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
table {
  width: 100%;
}
</style>
