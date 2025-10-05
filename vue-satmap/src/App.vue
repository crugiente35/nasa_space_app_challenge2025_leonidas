<template>
  <Sidebar
    @selection-changed="onSelectionChanged"
    @update-interval-changed="onUpdateIntervalChanged"
  />
  <SatelliteMap ref="map"/>
</template>

<script setup>
import { ref } from "vue";
import Sidebar from "./components/Sidebar.vue";
import SatelliteMap from "./components/SatelliteMap.vue";

const map = ref(null);

function onSelectionChanged({ name, enabled }) {
  if (enabled) {
    map.value.loadSatelliteGroup(name);
  } else {
    map.value.removeSatelliteGroup(name);
  }
}

function onUpdateIntervalChanged(interval) {
  map.value.setUpdateInterval(interval);
}
</script>