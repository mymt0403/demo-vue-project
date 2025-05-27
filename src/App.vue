<script setup>
import SearchItems from './SearchItems.vue'
import InfoDetailList from './InfoDetailList.vue'
import { ref, onMounted } from 'vue';
const tokyoLat = parseFloat(import.meta.env.VITE_DEFAULT_TOKYO_LAT)
const tokyoLng = parseFloat(import.meta.env.VITE_DEFAULT_TOKYO_LNG)
const handleFacilitiesData = ref([])

onMounted(async () => {
  const { Map } = await window.google.maps.importLibrary('maps');
  const map = new Map(document.getElementById('map'), {
    center: { lat: tokyoLat, lng: tokyoLng },
    zoom: 15
  });
});

/** 親コンポーネント経由でデータを渡す */
function handleData(payload) {
  handleFacilitiesData.value = payload
}
</script>

<template>
<div><h1>粗大ごみ持込可能施設 一覧</h1></div>
<SearchItems @sendData="handleData"/>
<div class="container">
    <div class="googleMap" id="map"></div>
    <div class="selectedFacilities">
      <InfoDetailList :facilities="handleFacilitiesData"/>
    </div>
</div>
</template>

<style>
h1 {
  margin-bottom: 10px;
}

.container {
  display: flex;
  width: 100%;
  height: 600px;
}

.googleMap {
  height: 100%;
  width: 65%;
}

.selectedFacilities {
  flex: 0 1 35%;
  height: 100%;
  padding: 10px;
  overflow: hidden;
}
</style>
