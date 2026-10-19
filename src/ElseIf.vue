<script setup lang="ts">
const isA = (o: { kind: string }) => o.kind === "a";
const show = false;
</script>

<template>
  <p v-if="show">shown</p>
  <p v-else-if="isA({ kind: 'a' })">reported</p>
</template>
