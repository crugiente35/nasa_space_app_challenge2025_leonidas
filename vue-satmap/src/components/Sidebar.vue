<template>
  <div class="sidebar">
    <h3>Satellite options</h3>
    <div class="option">
      <label>
        Update interval (seconds):
        <input type="number" min="1" v-model.number="updateInterval" @change="emitUpdateInterval" />
      </label>
    </div>
    <h3>Select constellation</h3>
    <ul>
      <li v-for="(link, name) in satelliteGroups" :key="name">
        <label>
          <input type="checkbox" v-model="selected[name]" @change="updateSelection(name)">
          {{ name }}
        </label>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from "vue";
import satelliteGroups from "./satellite-groups.json";

const selected = ref({});
for (const name in satelliteGroups) {
  selected.value[name] = false;
}

const updateInterval = ref(5); // default 5 seconds

const emit = defineEmits([
  "selection-changed",
  "update-interval-changed"
]);

function updateSelection(name) {
  emit("selection-changed", { name, enabled: selected.value[name] });
}
function emitUpdateInterval() {
  emit("update-interval-changed", updateInterval.value);
}
</script>

<style>
.sidebar {
  position: absolute;
  left: 0;
  top: 0;
  width: 220px;
  height: 100vh;
  background: rgba(0,0,0,0.8);
  color: white;
  padding: 10px;
  overflow-y: auto;
  z-index: 10;
}
.option {
  margin-bottom: 10px;
}
</style>