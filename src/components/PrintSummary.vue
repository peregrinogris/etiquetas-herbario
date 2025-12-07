<template>
  <div class="print-summary" v-if="labels.length > 0 && showSummary">
    <ol>
      <h1>Orden de entrega</h1>
      <li v-for="label in labels" :key="label.numero">
        <!-- Updated Name -->
        <i>{{ label.scientificName }}</i>{{ label.authors ? ' ' + label.authors : '' }}

        <!-- Old Name (Synonym) -->
        <span v-if="label.synonymName">
          (=<i>{{ label.synonymName }}</i>{{ label.synonymAuthors ? ' ' + label.synonymAuthors : '' }})
        </span>

        <!-- Number at end -->
        <span style="color: #666; font-size: 0.9em; margin-left: 5px;">#{{ label.numero }}</span>
      </li>
    </ol>
  </div>
</template>

<script setup>
defineProps({
  labels: Array,
  showSummary: Boolean,
})
</script>

<style scoped>
.print-summary {
  display: none; /* Hidden on screen */
  font-family: 'Segoe UI', sans-serif;
  max-width: 21cm;
  margin: 0 auto;
  break-after: page;
  page-break-after: always; /* Forces new page after list */
  width: 100%;
  box-sizing: border-box;
}

.print-summary h1 {
  font-size: 18pt;
  color: #000;
  break-after: avoid;
}

/* Column Layout Controlled by CSS Variable */
.print-summary ol {
  font-size: 10pt; /* Smaller font for checklist */
  line-height: 1.5;
  color: #000;
  column-width: var(--summary-col-width);
  column-gap: 1cm;
}

.print-summary li {
  margin-bottom: 2px;
  break-inside: avoid;
  page-break-inside: avoid;
}

@media print {
  .print-summary {
    display: block;
  }
}
</style>
