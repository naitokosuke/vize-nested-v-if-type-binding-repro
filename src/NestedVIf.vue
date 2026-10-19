<script setup lang="ts">
const isA = (o: { kind: string }) => o.kind === "a";
const show = true;
</script>

<template>
  <p v-if="isA({ kind: 'a' })">not reported</p>
  <div v-if="show">
    <p v-if="isA({ kind: 'a' })">reported</p>
  </div>
</template>
