<script setup>
import { defineProps } from 'vue';
const props = defineProps({
    facilities: {
        type: Array,
        default: () => []
    }
});
</script>

<template>
    <div class="resultContainer">
        <div class="list-title"><h3>選択した都道府県内の施設</h3></div>
        <div class="facility-list">
            <div v-if="!facilities.length">※対象の施設は見つかりませんでした。</div>
            <div v-for="(facility, i) in facilities" :key="i" class="facility-item">
                <div class="facility-name">{{ i + 1 }}. {{ facility.facilityName }} ({{ facility.garbageTypeName }})</div>
                <div>
                住所: {{ facility.address }}（
                <a :href="facility.mapUrl" target="_blank">GoogleMapで見る</a>）
                </div>
                <div>緯度: {{ facility.latitude }}, 経度: {{ facility.longitude }}</div>
                <hr />
            </div>
        </div>
    </div>
</template>

<style scoped>
.resultContainer {
    height: 100%;
    flex-direction: column;
}

h3 {
    margin-bottom: 10px;
}

.facility-list {
    flex-grow: 1;
    overflow-y: auto;
    padding: 10px;
}
</style>
