<script setup>
import ExcelConverter from "@/components/ExcelConverter.vue";
</script>

<template>
  <div id="app">
    <v-app>
      <v-card>
        <v-card-title>Excel converter</v-card-title>
        <v-card-text>
          <ExcelConverter />
        </v-card-text>
      </v-card>
    </v-app>
  </div>
</template>

<style>
#app {
  padding: 1rem;
  background-color: #eeeeee;
}
</style>
