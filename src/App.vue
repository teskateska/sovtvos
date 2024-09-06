<script setup lang="ts">
import KamalaCube from "./components/KamalaCube.vue";
</script>

<template>
  <KamalaCube />
</template>

<style scoped>
.content {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  color: blue; /* Adjust text color for visibility */
  padding: 20px;
}
</style>
