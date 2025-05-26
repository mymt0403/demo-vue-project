<template>
  <div id="map"></div>
</template>

<script setup>
import { onMounted } from 'vue';

onMounted(async () => {
  const { Map } = await window.google.maps.importLibrary('maps');

  const map = new Map(document.getElementById('map'), {
    center: { lat: 35.681236, lng: 139.767125 }, // 東京駅
    zoom: 15
  });
});
</script>

<style>
#map {
    width: 65%;
    height: 700px;
}
</style>
