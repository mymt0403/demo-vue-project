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
                <div class="facility-name">
                    <div class="circle">{{ i + 1 }}</div>
                    <div class="label">{{ facility.facilityName }} ({{ facility.garbageTypeName }})</div>
                </div>
                <div>住所: {{ facility.address }}（<a :href="facility.mapUrl" target="_blank">GoogleMapで見る</a>）</div>
                <div>緯度: {{ facility.latitude }}, 経度: {{ facility.longitude }}</div>
                <hr />
            </div>
        </div>
    </div>
</template>

<style scoped>
.resultContainer {
    height: 100%;
    display: flex;
    flex-direction: column;
    box-sizing: border-box;
}

h3 {
    margin-bottom: 10px;
    font-weight: bold;
}

.facility-list {
    flex-grow: 1;
    overflow-y: auto;
    height: calc(100% - 20px); /* padding分を引く */
    box-sizing: border-box; /* paddingも高さに含める */
}

.facility-name {
    display: flex;
    align-items: center;
    font-weight: bold;
    font-size: 1.2em;
    margin-top: 3px;
}

.circle {
    width: 25px;
    height: 25px;
    background-color: rgb(255, 65, 65);
    color: white;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 0.9em;
    flex-shrink: 0;
    font-weight: bold;
}

.label {
    color: rgb(255, 65, 65);
    font-weight: bold;
    margin-left: 5px;
}
</style>
