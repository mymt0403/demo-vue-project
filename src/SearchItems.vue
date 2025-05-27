<script setup>
import { ref, defineEmits, onMounted } from 'vue';
import axios from 'axios';

let map
const tokyoLat = parseFloat(import.meta.env.VITE_DEFAULT_TOKYO_LAT)
const tokyoLng = parseFloat(import.meta.env.VITE_DEFAULT_TOKYO_LNG)
const emit = defineEmits([`sendData`])
const selectedValue = ref('40')

async function fetchCenter() {
    const { Map } = await window.google.maps.importLibrary('maps');

    /** 中央座標を取得し、マップを表示する */
    await axios.get(`http://localhost:8080/api/center/${selectedValue.value}`)
        .then(
            function (response) {
                map = new Map(document.getElementById("map"), {
                    center: {
                        lat: response.data.latitude,
                        lng: response.data.longitude,
                    },
                    zoom: 9,
                    mapId: "4504f8b37365c3d0",
                })
            }
        )

        /** 選択した都道府県の施設情報を取得し、マップ上にピンを指す */
    await axios.get(`http://localhost:8080/api/data/${selectedValue.value}`)
        .then(
            function (response) {
                putPins(response.data)
            }
        )
}

async function putPins(pins) {
    const { AdvancedMarkerElement } = await google.maps.importLibrary("marker");
    pins.forEach((pin, i) => {
        new AdvancedMarkerElement({
            map: map,
            position: {
                lat: pin.latitude,
                lng: pin.longitude,
            },
            title: pin.facilityName,
            content: new google.maps.marker.PinElement({
                glyph: `${i + 1}`,
                glyphColor: "white",
                scale: 1,
            }).element,
        });
    });
    emit(`sendData`, pins)
}

function showUserLocation() {
    const { AdvancedMarkerElement } = google.maps.importLibrary("marker");
    const { PinElement } = google.maps.importLibrary("marker");

    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
            (position) => {
                const pos = {
                    lat: position.coords.latitude,
                    lng: position.coords.longitude,
                };

               /**  Font Awesome のアイコンを使用する */
                const icon = document.createElement("div");
                icon.innerHTML = '<i class="fa-solid fa-user"></i>';

                const faPin = new google.maps.marker.PinElement({
                    glyph: icon,
                    glyphColor: "#000000",
                    background: "#FF6633",
                    borderColor: "#FF0000",
                });

                /** 現在地をマークする */
                new google.maps.marker.AdvancedMarkerElement({
                    map,
                    position: pos,
                    content: faPin.element,
                    title: "your location",
                });

                map.setCenter(pos);
            },
            () => {
                handleLocationError();
            },
        );
    } else {
        handleLocationError();
    }
}

/** 初期表示時に検索を実行 */
onMounted(() => {
    fetchCenter()
})

function handleLocationError() {
    const errorMessageElement = document.getElementById("errorMessage");
    errorMessageElement.innerHTML = `
        <p>位置情報が取得できませんでした。<br>
        位置情報を表示させる場合はブラウザの設定から位置情報へのアクセスを許可してください。</p>`;
    errorMessageElement.style.display = "block";
}
</script>

<template>
    <div class="selectContainer">
        <select id="mySelect" v-model="selectedValue">
            <option value="40">福岡県</option>
            <option value="41">佐賀県</option>
            <option value="42">長崎県</option>
            <option value="43">熊本県</option>
            <option value="44">大分県</option>
            <option value="45">宮崎県</option>
            <option value="46">鹿児島県</option>
            <option value="47">沖縄県</option>
        </select>
        <button class="button1" @click="fetchCenter">search</button>
        <button class="button2" @click="showUserLocation">display your location</button>
    </div>
    <div class="error" style="vertical-align: top;">
        <div id="errorMessage" class="error-message"></div>
    </div>
</template>

<style>
.selectContainer {
    gap: 10px;
    width: 100%;
    max-width: 960px;
    display: flex;
    align-items: left;
    margin-bottom: 1em;
}

#mySelect {
    width: 150px;
    height: 40px;
    font-size: 16px;
}

.button1 {
    padding: 5px 10px;
    width: 100px;
    height: 40px;
    font-size: 16px;
}

.button2 {
    padding: 5px 10px;
    width: 200px;
    height: 40px;
    font-size: 16px;
}

.error-message {
    display: none;
    color: red;
    font-size: 14px;
    margin-bottom: 15px;
}
</style>
