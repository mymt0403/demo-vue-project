<script setup>
import SearchItems from './SearchItems.vue'
import InfoDetailList from './InfoDetailList.vue'
import { onMounted } from 'vue';

onMounted(async () => {
  const { Map } = await window.google.maps.importLibrary('maps');

  const map = new Map(document.getElementById('map'), {
    center: { lat: 35.681236, lng: 139.767125 }, // 東京駅
    zoom: 15
  });
});
</script>

<template>
<div><h1>粗大ごみ持込可能施設 一覧</h1></div>
<SearchItems />
<div class="container">
    <div class="googleMap" id="map"></div>
    <div class="selectedFacilities">
      <InfoDetailList />
    </div>
</div>
</template>

<style>
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
}
</style>
