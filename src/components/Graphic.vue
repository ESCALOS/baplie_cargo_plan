<script setup lang="ts">
import BayGrid from "./BayGrid.vue";

// Definir el tipo Container con la propiedad podLetter
interface Container {
  bay: string;
  row: number;
  tier: number;
  podLetter: string; // Letra del puerto de descarga
}

// Definir las props que recibe el componente
const props = defineProps<{ containers: Container[] }>();

// Identificar las bahías base (4xN + 2)
const isBaseBay = (bay: number) => (bay - 2) % 4 === 0;

// Agrupar contenedores por bahías relacionadas
const groupedBays = props.containers.reduce((acc, container) => {
  const bayNumber = parseInt(container.bay, 10);

  // Determinar la bahía base
  const baseBay = isBaseBay(bayNumber)
    ? bayNumber
    : Math.floor(bayNumber / 4) * 4 + 2;

  // Agrupar por bahía base
  if (!acc[baseBay]) {
    acc[baseBay] = [];
  }
  acc[baseBay].push(container);

  return acc;
}, {} as Record<number, Container[]>);

// Crear datos agrupados para renderizar
const groupedData = Object.entries(groupedBays).flatMap(
  ([baseBay, containers]) => {
    const baseBayNumber = parseInt(baseBay, 10);
    const hasLowerBay = containers.some(
      (container) => parseInt(container.bay, 10) === baseBayNumber - 1
    );
    const hasUpperBay = containers.some(
      (container) => parseInt(container.bay, 10) === baseBayNumber + 1
    );

    const result = [];

    // Gráfico para la bahía base y la bahía menor
    if (hasLowerBay) {
      result.push({
        title: `BAHÍA ${baseBayNumber - 1}/${baseBayNumber}`,
        containers: containers.filter(
          (container) =>
            parseInt(container.bay, 10) === baseBayNumber - 1 ||
            parseInt(container.bay, 10) === baseBayNumber
        ),
        maxRow: Math.max(...containers.map((c) => c.row)),
      });
    } else {
      result.push({
        title: `BAHÍA ${baseBayNumber}`,
        containers: containers.filter(
          (container) => parseInt(container.bay, 10) === baseBayNumber
        ),
        maxRow: Math.max(...containers.map((c) => c.row)),
      });
    }

    // Gráfico para la bahía mayor
    if (hasUpperBay) {
      result.push({
        title: `BAHÍA ${baseBayNumber + 1}`,
        containers: containers.map((container) => ({
          ...container,
          podLetter:
            parseInt(container.bay, 10) === baseBayNumber + 1
              ? container.podLetter
              : "", // Eliminar letras para la bahía base
        })),
        maxRow: Math.max(...containers.map((c) => c.row)),
      });
    }

    return result;
  }
);
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
