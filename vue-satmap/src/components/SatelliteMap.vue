<template>
  <div id="cesiumContainer" class="w-full h-screen"></div>
</template>

<script setup>
import { onMounted, ref, defineExpose }  from "vue";
import * as Cesium from "cesium";
import * as satellite from "satellite.js";
import satelliteGroups from "./satellite-groups.json";

const viewer = ref(null);
const updateInterval = ref(5); // seconds
let updateTimer = null;
const groupSatData = ref({}); 
const groupSatRaw = ref({});

function clearUpdateTimer() {
  if (updateTimer) {
    clearInterval(updateTimer);
    updateTimer = null;
  }
}

function startUpdateTimer() {
  clearUpdateTimer();
  updateTimer = setInterval(updateSatPositions, updateInterval.value * 1000);
}
function setUpdateInterval(interval) {
  updateInterval.value = interval;
  startUpdateTimer();
}


// Función para obtener la posición actual del satélite
function getSatellitePosition(tle1, tle2, date = new Date()) {
  const satrec = satellite.twoline2satrec(tle1, tle2);
  const pv = satellite.propagate(satrec, date);
  if (!pv.position) return null;

  const gmst = satellite.gstime(date);
  const gd = satellite.eciToGeodetic(pv.position, gmst);

  return {
    longitude: satellite.degreesLong(gd.longitude),
    latitude: satellite.degreesLat(gd.latitude),
    altitude: gd.height * 1000,
  };
}

// Elimina todos los satélites de un grupo del mapa
function clearSatEntities(groupName) {
  if (!groupSatData.value[groupName]) return;
  groupSatData.value[groupName].forEach(sat => {
    viewer.value.entities.remove(sat.entity);
  });
  groupSatData.value[groupName] = [];
}
// Devuelve true si la posición está dentro del rectángulo de la cámara
function isInView(pos, rectangle) {
  return (
    pos.longitude >= Cesium.Math.toDegrees(rectangle.west) &&
    pos.longitude <= Cesium.Math.toDegrees(rectangle.east) &&
    pos.latitude >= Cesium.Math.toDegrees(rectangle.south) &&
    pos.latitude <= Cesium.Math.toDegrees(rectangle.north)
  );
}


async function fetchSatellites(url) {
  const response = await fetch(url);
  if (!response.ok) throw new Error("Failed to download TLE");
  const tleText = await response.text();
  const lines = tleText.split("\n").filter(l => l.trim() !== "");
  const sats = [];
  for (let i = 0; i < lines.length; i += 3) {
    const name = lines[i].trim();
    const tle1 = lines[i + 1];
    const tle2 = lines[i + 2];
    if (!tle1 || !tle2) continue;
    sats.push({ name, tle1, tle2 });
  }
  return sats;
}


async function loadSatelliteGroup(groupName) {
  if (!viewer.value) return;
  if (!groupSatRaw.value[groupName]) {
    // Descarga y guarda todos los TLEs del grupo
    const url = satelliteGroups[groupName];
    if (!url) return;
    groupSatRaw.value[groupName] = await fetchSatellites(url);
  }
  updateSatellitesInView(groupName);
  startUpdateTimer();
}

// Muestra solo los satélites del grupo que están en la vista de la cámara
function updateSatellitesInView(groupName) {
  if (!viewer.value || !groupSatRaw.value[groupName]) return;
  clearSatEntities(groupName);

  const rectangle = viewer.value.camera.computeViewRectangle(viewer.value.scene.globe.ellipsoid);
  const now = new Date();
  groupSatData.value[groupName] = groupSatRaw.value[groupName]
    .map(sat => {
      const pos = getSatellitePosition(sat.tle1, sat.tle2, now);
      if (!pos || !isInView(pos, rectangle)) return null;
      const entity = viewer.value.entities.add({
        name: sat.name,
        position: Cesium.Cartesian3.fromDegrees(pos.longitude, pos.latitude, pos.altitude),
        point: { pixelSize: 4, color: Cesium.Color.CYAN },
        label: { text: sat.name, font: "10px sans-serif", fillColor: Cesium.Color.WHITE },
      });
      return { ...sat, entity };
    })
    .filter(Boolean);
}
function updateSatPositions() {
  for (const group in groupSatRaw.value) {
    updateSatellitesInView(group);
  }
}
// Escucha los movimientos de la cámara para recargar los satélites visibles
function setupCameraListener() {
  viewer.value.camera.moveEnd.addEventListener(() => {
    for (const group in groupSatRaw.value) {
      updateSatellitesInView(group);
    }
  });
}
function removeSatelliteGroup(groupName) {
  if (!groupSatData.value[groupName]) return;
  groupSatData.value[groupName].forEach(sat => {
    viewer.value.entities.remove(sat.entity);
  });
  delete groupSatData.value[groupName];
}
defineExpose({
  loadSatelliteGroup,
  removeSatelliteGroup,
  setUpdateInterval,
});
onMounted(() => {
  viewer.value = new Cesium.Viewer("cesiumContainer", {
    timeline: false,
    animation: false,
    baseLayerPicker: true,
  });
  viewer.value.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(0, 0, 3000000),
  });
  setupCameraListener();
  startUpdateTimer();
});
</script>

<style>
#cesiumContainer {
  width: 100%;
  height: 100vh;
  margin: 0;
  padding: 0;
}
</style>
