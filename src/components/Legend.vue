<script setup lang="ts">
interface LegendItem {
  letter: string;
  pod: string;
  color: string; // Color asignado
}

const props = defineProps<{
  legend: LegendItem[];
}>();
</script>

<template>
  <div v-if="props.legend.length" class="my-4 col-span-4">
    <h2 class="text-xl font-semibold mb-4">Leyenda</h2>
    <div class="flex flex-wrap gap-4">
      <div
        v-for="item in props.legend"
        :key="item.letter"
        class="flex items-center rounded-lg"
        :style="{ backgroundColor: item.color }"
      >
        <!-- Letra -->
        <div class="text-gray-900 font-bold px-4 py-2">
          {{ item.letter }}
        </div>
        <!-- Separador -->
        <div class="border-l border-gray-900 h-full"></div>
        <!-- Puerto -->
        <div class="flex-1 px-4 py-2 text-gray-900">
          {{ item.pod }}
        </div>
      </div>
    </div>
  </div>
</template>
