<template>
  <div>
    <h1 class="text-2xl font-bold mb-4">Cargar Archivo Excel</h1>

    <!-- Pantalla de carga -->
    <div
      v-if="loading"
      class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50"
    >
      <div class="text-center">
        <div class="loader mb-4"></div>
        <p class="text-white">Procesando archivo, por favor espere...</p>
      </div>
    </div>

    <!-- Formulario para cargar archivo -->
    <form v-else>
      <label class="block mb-2 text-sm font-medium text-gray-700" for="file">
        Selecciona un archivo Excel:
      </label>
      <input
        id="file"
        type="file"
        accept=".xlsx, .xls"
        class="block w-full text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50 focus:outline-none"
        @change="handleFileUpload"
      />
    </form>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import * as XLSX from "xlsx";

const emit = defineEmits(["dataProcessed"]);
const loading = ref(false); // Estado de carga

const handleFileUpload = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (!file) return;

  loading.value = true; // Mostrar pantalla de carga

  const reader = new FileReader();
  reader.onload = (e) => {
    const data = new Uint8Array(e.target?.result as ArrayBuffer);
    const workbook = XLSX.read(data, { type: "array" });

    // Leer la primera hoja del archivo Excel
    const sheetName = workbook.SheetNames[0];
    const sheet = workbook.Sheets[sheetName];

    // Convertir los datos de la hoja a JSON
    const jsonData = XLSX.utils.sheet_to_json(sheet);

    // Procesar los datos
    const processedData = processExcelData(jsonData);

    // Emitir el evento con los datos procesados
    emit("dataProcessed", processedData);

    loading.value = false; // Ocultar pantalla de carga
  };
  reader.readAsArrayBuffer(file);
};

const processExcelData = (data: any[]) => {
  const containers = [];
  const podSet = new Set<string>();

  for (const row of data) {
    const bay = row["BAHIA"];
    const rowNumber = row["ROW"];
    const tier = row["TIER"];
    const pod = row["POD"];

    if (bay && rowNumber && tier && pod) {
      containers.push({
        bay: parseInt(bay),
        row: parseInt(rowNumber),
        tier: parseInt(tier),
        pod,
      });
      podSet.add(pod);
    }
  }

  // Crear la leyenda
  const legend = Array.from(podSet)
    .sort()
    .map((pod, index) => ({
      letter: String.fromCharCode(65 + index), // A, B, C, ...
      pod,
    }));

  return { containers, legend };
};
</script>

<style>
/* Loader CSS */
.loader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

/* Fondo con opacidad */
.bg-opacity-50 {
  background-color: rgba(0, 0, 0, 0.5);
}
</style>
