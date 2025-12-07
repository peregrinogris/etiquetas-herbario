<template>
  <div class="settings-panel">
    <div class="setting-item">
      <label>Tipografía</label>
      <select v-model="settings.fontFamily">
        <option value="'EB Garamond', serif">EB Garamond</option>
        <option value="'PT Serif', serif">PT Serif</option>
        <option value="'Courier Prime', monospace">Courier Prime</option>
      </select>
    </div>
    <div class="setting-item">
      <label>Tamaño Fuente (px)</label>
      <input type="number" v-model="settings.fontSize" step="1">
    </div>
    <div class="setting-item">
      <label>Ancho (cm)</label>
      <input type="number" v-model="settings.width" step="0.5">
    </div>
    <div class="setting-item">
      <label>Alto (cm)</label>
      <input type="number" v-model="settings.height" step="0.5">
    </div>
    <div class="setting-item">
      <label>Relleno (cm)</label>
      <input type="number" v-model="settings.padding" step="0.1">
    </div>
    <div class="setting-item">
      <label>Grosor Borde (px)</label>
      <input type="number" v-model="settings.borderWidth" step="1">
    </div>
    <div class="setting-item">
      <label>Estilo Borde</label>
      <select v-model="settings.borderStyle">
        <option value="solid">Sólido</option>
        <option value="dashed">Discontinuo</option>
        <option value="dotted">Punteado</option>
        <option value="double">Doble</option>
        <option value="none">Ninguno</option>
      </select>
    </div>
    <div class="setting-item restore-button-container">
      <button class="btn-outline" style="width: 100%; font-size: 12px;" @click="resetSettings">Restaurar</button>
    </div>
  </div>

  <div class="preview-container">
    <Label :label="sampleLabel" :collector="sampleCollector" />
  </div>
</template>

<script setup>
import Label from './Label.vue'; // Import the Label component
import { ref } from 'vue';

defineProps({
  settings: Object,
  resetSettings: Function,
})

// Sample data for the Label preview
const sampleLabel = ref({
  numero: '12',
  familia: 'Lamiaceae',
  scientificName: 'Salvia officinalis',
  authors: 'L.',
  synonymName: null,
  synonymAuthors: null,
  vulgar: 'Salvia, Salvia común',
  fechaLugar: '15 Mayo 2023. Jardín Botánico',
  obs: 'Planta de unos 50 cm de alto, con flores violáceas.'
});

const sampleCollector = ref('Carlos Linneo');
</script>

<style scoped>
.settings-panel {
  background: #f8f9fa;
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 15px;
  animation: slideDown 0.2s ease-out;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.setting-item {
  display: flex;
  flex-direction: column;
}

.restore-button-container {
  justify-content: flex-end;
}

label {
  font-weight: 600;
  display: block;
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

input, select {
  width: 100%;
  padding: 10px;
  border: 1px solid var(--border);
  border-radius: 6px;
  box-sizing: border-box;
  font-family: inherit;
}

.btn-outline {
  background: transparent;
  border: 1px solid var(--border);
  color: var(--primary);
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: transform 0.1s;
}

.preview-container {
  margin-top: 2rem;
  padding-top: 1rem;
  display: flex;
  justify-content: center;
}
</style>
