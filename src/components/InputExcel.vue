<template>
  <h1 class="text-2xl font-bold mb-4">Cargar Archivo Excel</h1>
  <form>
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
</template>

<script setup lang="ts">
import * as XLSX from "xlsx";

const emit = defineEmits(["dataProcessed"]);

const handleFileUpload = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (!file) return;

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
    emit("dataProcessed", processedData);
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
        bay,
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
