<script setup lang="ts">
import BayGrid from "./BayGrid.vue";
import Legend from "./Legend.vue";
import { computed } from "vue";

interface Container {
  bay: number;
  row: number;
  tier: number;
  podLetter: string;
}

const props = defineProps<{
  containers: Container[];
  podColorMap: Map<string, string>;
  legend: {
    letter: string;
    pod: string;
    color: string;
  }[];
}>();

const isBaseBay = (bay: number) => bay % 4 === 2;

const formatThreeDigits = (value: number) => value.toString().padStart(3, "0");

const groupedBays = props.containers.reduce((acc, container) => {
  const bay = container.bay;
  if (!acc[bay]) acc[bay] = [];
  acc[bay].push(container);
  return acc;
}, {} as Record<number, Container[]>);

const groupedData = Object.keys(groupedBays)
  .map(Number)
  .filter(isBaseBay)
  .flatMap((baseBay) => {
    const lowerBay = baseBay - 1; // 4N+1
    const upperBay = baseBay + 1; // 4N+3

    const hasLowerBay = groupedBays[lowerBay];
    const hasBaseBay = groupedBays[baseBay];
    const hasUpperBay = groupedBays[upperBay];

    const maxRow = 20;
    const result = [];

    if (hasLowerBay && hasBaseBay) {
      result.push({
        title: `${formatThreeDigits(lowerBay)} / ${formatThreeDigits(baseBay)}`,
        containers: [...groupedBays[lowerBay], ...groupedBays[baseBay]],
        maxRow,
      });
    } else if (hasBaseBay) {
      result.push({
        title: `${formatThreeDigits(baseBay)}`,
        containers: [...groupedBays[baseBay]],
        maxRow,
      });
    }

    if (hasUpperBay) {
      const baseWithoutLetters =
        groupedBays[baseBay]?.map((c) => ({
          ...c,
        })) ?? [];
      result.push({
        title: `${formatThreeDigits(upperBay)}`,
        containers: [...baseWithoutLetters, ...groupedBays[upperBay]],
        maxRow,
      });
    }

    return result;
  });

const groupedDataByPage = computed(() => {
  const groups = [];
  for (let i = 0; i < groupedData.length; i += 8) {
    groups.push(groupedData.slice(i, i + 8)); // Agrupa de 8 en 8
  }
  return groups;
});
</script>

<template>
  <div>
    <template
      v-for="(group, groupIndex) in groupedDataByPage"
      :key="groupIndex"
    >
      <!-- Contenedor para cada hoja A4 -->
      <div class="a4-container">
        <div class="grid grid-cols-4 gap-2">
          <!-- Leyenda para cada grupo -->
          <Legend :legend="legend" class="col-span-4 mb-4" />
          <!-- Bahías dentro del grupo -->
          <template v-for="bayData in group" :key="bayData.title">
            <div>
              <BayGrid
                :bay="bayData.title"
                :maxRow="bayData.maxRow"
                :containers="bayData.containers"
                :podColorMap="podColorMap"
              />
            </div>
          </template>
        </div>
      </div>
    </template>
  </div>
</template>
