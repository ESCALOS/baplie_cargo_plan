<script setup lang="ts">
interface Container {
  bay: string; // Número de la bahía a la que pertenece el contenedor
  row: number;
  tier: number;
  podLetter: string; // Letra del puerto de descarga
}

const props = defineProps<{
  bay: string; // Título de la bahía (puede ser "BAHÍA 001/002")
  maxRow: number;
  containers: Container[];
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
  ...Array.from({ length: 9 }, (_, i) => 18 - i * 2), // Parte inferior (18 al 02)
];

// Formatear los valores de ROW y TIER para que tengan dos dígitos
const formatTwoDigits = (value: number) => value.toString().padStart(2, "0");
</script>

<template>
  <div class="mb-8">
    <h2 class="text-lg font-bold mb-4">{{ bay }}</h2>
    <div class="overflow-auto">
      <table class="table-auto border-collapse border border-gray-300">
        <thead>
          <tr>
            <th class="border border-gray-300 p-2 bg-gray-100"></th>
            <th
              v-for="row in rows"
              :key="row"
              class="border border-gray-300 p-2 bg-gray-100 text-center"
            >
              {{ formatTwoDigits(row) }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="tier in tiers" :key="tier">
            <td class="border border-gray-300 p-2 bg-gray-100 text-center">
              {{ formatTwoDigits(tier) }}
            </td>
            <td
              v-for="row in rows"
              :key="row"
              class="border border-gray-300 p-2 text-center"
              :class="{
                'bg-blue-500 text-white': containers.some(
                  (container) =>
                    container.row === row && container.tier === tier
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
                  containers.find(
                    (container) =>
                      container.row === row && container.tier === tier
                  )?.podLetter
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
