<script setup lang="ts">
import pdfView from "../components/pdfviewer/pdfviewer.vue";
import { onMounted, ref } from "vue";
const pdfSrc = ref("");
onMounted(() => {
  pdfSrc.value = "./test.pdf";
});
</script>
<template>
  <div>
    <pdf-view :pdf="pdfSrc" />
  </div>
</template>
<style scoped></style>
