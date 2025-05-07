<script setup lang="ts">
import BayGrid from "./BayGrid.vue";

interface Container {
  bay: number;
  row: number;
  tier: number;
  podLetter: string;
}

const props = defineProps<{ containers: Container[] }>();

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

    const maxRow = Math.max(
      ...(groupedBays[lowerBay]?.map((c) => c.row) ?? []),
      ...(groupedBays[baseBay]?.map((c) => c.row) ?? []),
      ...(groupedBays[upperBay]?.map((c) => c.row) ?? [])
    );

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
          podLetter: "",
        })) ?? [];
      result.push({
        title: `${formatThreeDigits(upperBay)}`,
        containers: [...baseWithoutLetters, ...groupedBays[upperBay]],
        maxRow,
      });
    }

    return result;
  });
</script>

<template>
  <div>
    <div v-for="bayData in groupedData" :key="bayData.title">
      <BayGrid
        :bay="bayData.title"
        :maxRow="bayData.maxRow"
        :containers="bayData.containers"
      />
    </div>
  </div>
</template>
