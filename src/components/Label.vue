<template>
  <div class="label-card">
    <div class="header-row">
      <span><span class="label-key">Colector:</span> {{ collector }}</span>
      <span><span class="label-key">Nº:</span> {{ label.numero }}</span>
    </div>
    <div class="label-row"><span class="label-key">Familia:</span> {{ label.familia }}</div>

    <div class="label-row">
      <span class="label-key">Nombre Científico: </span>
      <span>
        <!-- Main Name (Updated or Original) -->
        <i>{{ label.scientificName }}</i>{{ label.authors ? ' ' + label.authors : '' }}
        <!-- Synonym (Old Name) if updated exists -->
        <span v-if="label.synonymName">
          (=<i>{{ label.synonymName }}</i>{{ label.synonymAuthors ? ' ' + label.synonymAuthors : '' }})
        </span>
      </span>
    </div>

    <div class="label-row"><span class="label-key">Nombre Vulgar:</span> {{ label.vulgar }}</div>
    <div class="label-row"><span class="label-key">Fecha y Lugar:</span> {{ label.fechaLugar }}</div>
    <div class="label-row"><span class="label-key">Observación:</span> {{ label.obs }}</div>
  </div>
</template>

<script setup>
defineProps({
  label: Object,
  collector: String,
})
</script>

<style scoped>
.label-card {
  background: white;
  width: var(--lbl-width);
  height: var(--lbl-height);
  padding: var(--lbl-padding);
  border-width: var(--lbl-border-w);
  border-style: var(--lbl-border-s);
  border-color: #000;
  font-family: var(--lbl-font);
  font-size: var(--lbl-font-size);
  box-sizing: border-box;
  line-height: 1.2;
  overflow: hidden;
  position: relative;
}

.label-key {
  font-weight: 700;
  color: #000;
}

.header-row {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 5px;
}

.label-row {
  margin-bottom: 5px;
}
</style>
