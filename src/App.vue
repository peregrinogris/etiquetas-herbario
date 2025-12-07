<template>
  <div id="app">
    <div class="controls">
      <!-- App Icon -->
      <div style="text-align: center; margin-bottom: 10px;">
        <img src="./icon.png" alt="Logo" style="width: 80px; height: auto;">
      </div>

      <div class="header-title">
        <h1>Generador de Etiquetas para Herbario</h1>
        <button class="btn-outline" @click="showSettings = !showSettings">
          {{ showSettings ? '✖ Cerrar' : '⚙️ Configuración' }}
        </button>
      </div>

      <!-- SETTINGS PANEL -->
      <Settings v-if="showSettings" :settings="settings" :reset-settings="resetSettings" />

      <!-- PERSISTENT COLLECTOR INPUT -->
      <div class="input-group">
        <label>Colector</label>
        <input type="text" v-model="globalCollector" placeholder="Nombre del Colector">
      </div>

      <!-- DATA INPUT AREA -->
      <div class="input-group">
        <label>Importar Datos:</label>
        <div style="font-size: 0.85em; color: #666; margin-bottom: 5px;">
          Columnas: <strong>Número | Familia | Nombre Científico | Nombre Vulgar | Fecha | Lugar | Observación | Nombre Científico Actualizado (Opcional)</strong>
        </div>

        <!-- Tab 1: Paste -->
        <textarea
            v-if="inputMode === 'paste'"
            v-model="pasteInput"
            placeholder="Pegar celdas desde Excel/Google Sheets aquí..."
        ></textarea>

        <!-- Tab 2: Drag & Drop File -->
        <div
            v-else
            class="drop-zone"
            :class="{ 'drag-over': isDragging }"
            @dragover.prevent="isDragging = true"
            @dragleave.prevent="isDragging = false"
            @drop.prevent="handleDrop"
            @click="triggerFileSelect"
        >
          <p>Arrastra tu CSV aquí o haz clic para buscar</p>
          <input ref="fileInput" type="file" class="hidden-input" @change="handleFileSelect" accept=".csv">
        </div>

        <!-- Input Mode Toggles -->
        <div class="btn-group">
          <button class="btn-primary" @click="processData" :disabled="!hasData">Generar Etiquetas</button>
          <button class="btn-outline" @click="toggleMode">
            {{ inputMode === 'paste' ? 'Usar Archivo CSV' : 'Pegar Texto' }}
          </button>
          <button class="btn-outline" @click="clear" v-if="labels.length > 0">Limpiar</button>
        </div>
      </div>

      <!-- MAPPING DIAGNOSTICS -->
      <div class="status-panel" v-if="labels.length > 0">
        <div class="status-item" v-for="(found, col) in columnStatus" :key="col">
          <span class="status-dot" :class="found ? 'found' : 'missing'"></span>
          <span style="text-transform: capitalize;">{{ col }}</span>
        </div>
      </div>

      <!-- PRINT ACTIONS ROW -->
      <div v-if="labels.length > 0"
           style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px; border-top: 1px solid var(--border); padding-top: 20px;">
        <label
            style="display: flex; align-items: center; cursor: pointer; font-weight: normal; text-transform: none; font-size: 14px; margin: 0;">
          <input type="checkbox" v-model="showSummary" style="width: auto; margin-right: 8px;">
          Incluir lista "Orden de entrega"
        </label>
        <button class="btn-secondary" style="width: auto; padding-left: 30px; padding-right: 30px;" @click="print">
          🖨️ Imprimir
        </button>
      </div>
    </div>

    <!-- PRINT SUMMARY PAGE (Hidden on Screen, dependent on checkbox) -->
    <PrintSummary :labels="labels" :show-summary="showSummary"/>

    <!-- LABELS OUTPUT -->
    <div class="labels-grid">
      <Label v-for="(label, index) in labels" :key="index" :label="label" :collector="globalCollector"/>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import Label from './components/Label.vue';
import Settings from './components/Settings.vue';
import PrintSummary from './components/PrintSummary.vue';

// --- STATE ---
const globalCollector = ref('');
const pasteInput = ref('');
const labels = ref([]);
const inputMode = ref('paste'); // 'paste' | 'file'
const isDragging = ref(false);
const fileInput = ref(null);
const showSettings = ref(false);
const showSummary = ref(false); // Default disabled

// Settings State with Defaults
const defaultSettings = {
  width: 10,
  height: 7,
  padding: 0.5,
  borderWidth: 1,
  borderStyle: 'solid',
  fontFamily: "'EB Garamond', serif",
  fontSize: 16
};
const settings = ref({ ...defaultSettings });

// Diagnostic state to show user which columns were found
const columnStatus = ref({
  numero: false, familia: false, cientifico: false,
  cientificoAct: false,
  vulgar: false, fecha: false, lugar: false, obs: false
});

// --- PERSISTENCE ---
onMounted(() => {
  // Load Collector
  const savedCol = localStorage.getItem('label_collector');
  if (savedCol) globalCollector.value = savedCol;

  // Load Settings
  const savedSettings = localStorage.getItem('label_settings');
  if (savedSettings) {
    try {
      settings.value = { ...defaultSettings, ...JSON.parse(savedSettings) };
    } catch (e) {
      console.error("Error loading settings", e);
    }
  }

  // Initial Root Style Update
  updateRootStyles();
});

watch(globalCollector, (newVal) => {
  localStorage.setItem('label_collector', newVal);
});

// Deep watch on settings to save & update CSS variables
watch(settings, (newVal) => {
  localStorage.setItem('label_settings', JSON.stringify(newVal));
  updateRootStyles();
}, { deep: true });

// Watch labels to update dynamic summary layout
watch(labels, () => {
  updateRootStyles();
});

// --- COMPUTED ---
const hasData = computed(() => {
  if (inputMode.value === 'paste') return pasteInput.value.length > 5;
  return false; // File mode auto-processes on drop
});

// --- ACTIONS ---
const updateRootStyles = () => {
  const s = settings.value;
  const root = document.documentElement;

  // Dynamic Summary Width Calculation
  const count = labels.value.length;
  let summaryWidth = '10cm'; // Default < 25
  if (count > 50) summaryWidth = '5cm';
  else if (count >= 25) summaryWidth = '8cm';

  // Batch update via cssText on root element
  root.style.cssText = `
                        --lbl-width: ${s.width}cm;
                        --lbl-height: ${s.height}cm;
                        --lbl-padding: ${s.padding}cm;
                        --lbl-border-w: ${s.borderWidth}px;
                        --lbl-border-s: ${s.borderStyle};
                        --lbl-font: ${s.fontFamily};
                        --lbl-font-size: ${s.fontSize}px;
                        --summary-col-width: ${summaryWidth};
                    `;
};

const print = () => window.print();
const toggleMode = () => inputMode.value = inputMode.value === 'paste' ? 'file' : 'paste';
const clear = () => {
  labels.value = [];
  pasteInput.value = '';
};
const triggerFileSelect = () => fileInput.value.click();
const resetSettings = () => settings.value = { ...defaultSettings };

// --- FILE HANDLERS ---
const handleFileSelect = (e) => processFile(e.target.files[0]);
const handleDrop = (e) => {
  isDragging.value = false;
  processFile(e.dataTransfer.files[0]);
};

const processFile = (file) => {
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (e) => {
    const content = e.target.result;
    pasteInput.value = content;
    inputMode.value = 'paste'; // Switch view to textarea
    parseAndGenerate(content, ','); // Auto-trigger generation
  };
  reader.readAsText(file);
};

const processData = () => {
  if (inputMode.value === 'paste') parseAndGenerate(pasteInput.value, '\t');
};

// --- PARSING CORE ---
const parseAndGenerate = (rawText, fallbackDelimiter) => {
  const lines = rawText.split(/\r\n|\n/);
  if (lines.length < 2) return alert("El archivo parece vacío o sin cabeceras");

  // 1. Detect Delimiter
  let delimiter = fallbackDelimiter;
  if (lines[0].indexOf('\t') > -1) delimiter = '\t';
  else if (lines[0].indexOf(',') > -1) delimiter = ',';

  // 2. Map Headers
  const headers = lines[0].split(delimiter).map(h => h.trim());
  const map = {
    numero: findIndex(headers, 'Número'),
    familia: findIndex(headers, 'Familia'),
    cientifico: findIndex(headers, 'Nombre Científico'),
    cientificoAct: findIndex(headers, 'Nombre Científico Actualizado'),
    vulgar: findIndex(headers, 'Nombre Vulgar'),
    fecha: findIndex(headers, 'Fecha'),
    lugar: findIndex(headers, 'Lugar'),
    obs: findIndex(headers, 'Observación')
  };

  // 3. Update Status Diagnostics
  Object.keys(map).forEach(k => columnStatus.value[k] = map[k] !== -1);

  // 4. Transform Data
  labels.value = lines.slice(1).map(line => {
    if (!line.trim()) return null;
    const d = parseCSVRow(line, delimiter);

    const fullSciName = safeGet(d, map.cientifico);
    const updatedSciName = safeGet(d, map.cientificoAct); // Get Updated Name

    // Determine Primary (Updated or Old) and Secondary (Old or null)
    let primary, secondary;
    if (updatedSciName) {
      primary = splitSciName(updatedSciName);
      secondary = splitSciName(fullSciName); // Old name becomes secondary
    } else {
      primary = splitSciName(fullSciName);
      secondary = null;
    }

    return {
      numero: safeGet(d, map.numero),
      familia: safeGet(d, map.familia),
      // Primary Name Fields
      scientificName: primary.name,
      authors: primary.authors,
      // Secondary Name Fields (Synonym)
      synonymName: secondary ? secondary.name : null,
      synonymAuthors: secondary ? secondary.authors : null,

      vulgar: safeGet(d, map.vulgar),
      fechaLugar: [safeGet(d, map.fecha), safeGet(d, map.lugar)].filter(Boolean).join('. '),
      obs: safeGet(d, map.obs)
    };
  })
      .filter(Boolean)
      .sort((a, b) => {
        // First sort by family
        const famDiff = (a.familia || '').localeCompare(b.familia || '');
        if (famDiff !== 0) return famDiff;

        // Then do full sort inside the family
        return a.scientificName.localeCompare(b.scientificName);
      });
};

// --- UTILS ---
const findIndex = (headers, key) => headers.findIndex(h => h.toLowerCase().includes(key.toLowerCase()));
const safeGet = (arr, idx) => (idx === -1 || !arr[idx]) ? '' : arr[idx];

const splitSciName = (fullName) => {
  if (!fullName) return { name: '', authors: '' };
  const words = fullName.split(' ');
  if (words.length <= 2) return { name: fullName, authors: '' };
  return {
    name: words.slice(0, 2).join(' '),
    authors: words.slice(2).join(' ')
  };
};

const parseCSVRow = (text, delimiter) => {
    const d = delimiter === '\t' ? '\\t' : ',';
    const re = new RegExp(`(?:\\s*"([^"]*)"\\s*|([^${d}"]*))(?:${d}|$)`, 'g');
    const a = [];
    text.replace(re, (m0, m1, m2) => {
        if (m0 === "" && a.length > 0) return '';
        if (m1 !== undefined) a.push(m1.replace(/""/g, '"'));
        else if (m2 !== undefined) a.push(m2.trim());
        return '';
    });
    if (new RegExp(`${d}\\s*$`).test(text)) a.push('');
    return a;
};
</script>

<style>
/* --- CORE VARIABLES --- */
:root {
  --primary: #2c3e50;
  --accent: #27ae60;
  --bg: #ecf0f1;
  --surface: #ffffff;
  --border: #bdc3c7;
  --error: #e74c3c;

  /* Default Label Variables */
  --lbl-width: 10cm;
  --lbl-height: 7cm;
  --lbl-padding: 0.5cm;
  --lbl-border-w: 1px;
  --lbl-border-s: solid;
  --lbl-font: 'EB Garamond', serif;
  --lbl-font-size: 16px;

  /* Dynamic Summary Variables */
  --summary-col-width: 10cm;
}

body {
  font-family: 'Segoe UI', system-ui, sans-serif;
  background-color: var(--bg);
  padding: 20px;
  color: var(--primary);
  margin: 0;
}

/* --- CONTROLS UI --- */
.controls {
  background: var(--surface);
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  margin: 0 auto 2rem auto;
  max-width: 800px;
}

.header-title {
  margin-top: 0;
  color: var(--primary);
  border-bottom: 2px solid var(--bg);
  padding-bottom: 1rem;
  margin-bottom: 1.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.input-group {
  margin-bottom: 1.5rem;
}

label {
  font-weight: 600;
  display: block;
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

input[type="text"], input[type="number"], select, textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid var(--border);
  border-radius: 6px;
  box-sizing: border-box;
  font-family: inherit;
}

textarea {
  height: 100px;
  font-family: 'Consolas', monospace;
  font-size: 13px;
  resize: vertical;
}

/* --- DRAG & DROP ZONE --- */
.drop-zone {
  border: 2px dashed var(--border);
  border-radius: 8px;
  padding: 2rem;
  text-align: center;
  transition: all 0.2s ease;
  cursor: pointer;
  background: #fafafa;
}

.drop-zone:hover, .drop-zone.drag-over {
  border-color: var(--accent);
  background: #f0f9f4;
}

.drop-zone p {
  margin: 0;
  color: #7f8c8d;
}

.hidden-input {
  display: none;
}

/* --- BUTTONS --- */
.btn-group {
  display: flex;
  gap: 10px;
  margin-top: 1rem;
}

button {
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  border: none;
  transition: transform 0.1s;
}

button:active {
  transform: translateY(1px);
}

.btn-primary {
  background-color: var(--accent);
  color: white;
}

.btn-secondary {
  background-color: var(--primary);
  color: white;
}

.btn-outline {
  background: transparent;
  border: 1px solid var(--border);
  color: var(--primary);
}

.btn-icon {
  background: transparent;
  border: none;
  font-size: 1.2rem;
  padding: 5px;
  color: var(--primary);
}

/* --- MAPPING STATUS DIAGNOSTIC --- */
.status-panel {
  background: #f8f9fa;
  border-radius: 6px;
  padding: 10px;
  margin-top: 15px;
  font-size: 0.85rem;
  display: flex;
  justify-content: space-between
}

.status-item {
  display: flex;
  align-items: center;
  gap: 5px;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #ddd;
}

.status-dot.found {
  background: var(--accent);
}

.status-dot.missing {
  background: var(--error);
}

/* --- LABEL GRID --- */
.labels-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
}

/* --- PRINT MEDIA --- */
@media print {
  html, body {
    height: auto;
    overflow: visible;
  }

  body {
    background: white;
    padding: 0;
    margin: 0;
  }

  .controls {
    display: none;
  }

  .labels-grid {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-content: center;
    min-height: 100vh; /* Only vertically center labels if they take up the whole page */
    margin: 0 auto;
  }

  .label-card {
    margin: 0.2cm;
    page-break-inside: avoid;
    break-inside: avoid;
  }
}
</style>