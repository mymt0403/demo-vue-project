<script setup>
import { ref, defineEmits, onMounted } from 'vue';
import axios from 'axios';

let map, userLocationMarker
const emit = defineEmits([`sendData`])
const selectedValue = ref('40')
const url = 'https://routes.googleapis.com/distanceMatrix/v2:computeRouteMatrix'
const apiKey = import.meta.env.VITE_GOOGLE_MAPS_API_KEY
const userPosition = {}

async function init() {
    const facilities = await fetchFacilitiesInfo()
    await putPins(facilities)
    putDragablePin()
    emit(`sendData`, facilities)
}

function change() {
    userPosition.lat = ""
    userPosition.lng = ""
}

async function search() {
    const facilities = await fetchFacilitiesInfo()
    await putPins(facilities)
    putDragablePin()
    const enrichedFacilities = await getRouteInfo(userPosition, facilities)
    emit(`sendData`, enrichedFacilities)
}

async function fetchFacilitiesInfo() {
    const { Map } = await window.google.maps.importLibrary('maps');

    /** 中央座標を取得し、マップを表示する */
    await axios.get(`http://localhost:8080/api/center/${selectedValue.value}`)
        .then(
            function (response) {
                if(!userPosition.lat && !userPosition.lng) {
                    userPosition.lat = response.data.latitude
                    userPosition.lng = response.data.longitude
                }

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
    const facilitiesRes = await axios.get(`http://localhost:8080/api/data/${selectedValue.value}`)
    return facilitiesRes.data
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
}

function displayUserLocation() {
    /**  Font Awesome のアイコンを使用する */
    const icon = document.createElement("div");
    icon.innerHTML = '<i class="fa-solid fa-user"></i>';

    const faPin = new google.maps.marker.PinElement({
        glyph: icon,
        glyphColor: "#FFFFFF",
        background: "#0000FF",
        borderColor: "#FFFFFF",
    });

    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
            (position) => {
                const errorMessageElement = document.getElementById("errorMessage")
                errorMessageElement.style.display = "none"

                // const userPosition = {
                //     lat: position.coords.latitude,
                //     lng: position.coords.longitude,
                // };

                /** 仮の現在地 */
                userPosition.lat = 33.747585595093774
                userPosition.lng = 130.64076497636538

                userLocationMarker.position = null

                /** 現在地をマークする */
                userLocationMarker = new google.maps.marker.AdvancedMarkerElement({
                    map,
                    position: userPosition,
                    gmpDraggable: true,
                    content: faPin.element,
                    title: "your location",
                });

                map.setCenter(userPosition);
            },
            () => {
                handleLocationError()
            },
        );
    } else {
        handleLocationError()
    }
}

/** 初期表示時に検索を実行 */
onMounted(() => {
    init()
})

async function getRouteInfo(position, pins) {
    if (!pins.length) return [];

    const payload = {
        origins: [
            {
                waypoint: {
                    location: {
                        latLng: {
                            latitude: position.lat,
                            longitude: position.lng
                        }
                    }
                },
            },
        ],
        destinations: pins.map(dest => ({
            waypoint: {
                location: {
                    latLng: {
                        latitude: dest.latitude,
                        longitude: dest.longitude
                    }
                }
            }
        })),
        travelMode: 'DRIVE',
        routingPreference: 'TRAFFIC_AWARE'
    }

    const headers = {
        'Content-Type': 'application/json',
        'X-Goog-Api-Key': apiKey,
        'X-Goog-FieldMask': 'originIndex,destinationIndex,duration,distanceMeters,status,condition'
    }

    const response = await axios.post(url, payload, { headers })
    return pins.map((pin, index) => {
        const match = response.data.find(d => d.destinationIndex === index);
        return {
            ...pin,
            duration: match ? match.duration : null,
            distanceMeters: match ? match.distanceMeters : null,
        };
    });
}

function putDragablePin () {
    const infoWindow = new google.maps.InfoWindow();

    /**  Font Awesome のアイコンを使用する */
    const icon = document.createElement("div");
    icon.innerHTML = '<i class="fa-solid fa-user"></i>';

    const faPin = new google.maps.marker.PinElement({
        glyph: icon,
        glyphColor: "#FFFFFF",
        background: "#0000FF",
        borderColor: "#FFFFFF",
    });

    userLocationMarker = new google.maps.marker.AdvancedMarkerElement({
        map,
        position: { lat: userPosition.lat, lng: userPosition.lng },
        gmpDraggable: true,
        content: faPin.element,
        title: "This marker is draggable.",
    });
    infoWindow.setContent(`出発地点まで動かして「search」を押してください。`);
    infoWindow.open(userLocationMarker.map, userLocationMarker);

    userLocationMarker.addListener("dragstart", (event) => {
        infoWindow.close();
    });

    userLocationMarker.addListener("dragend", (event) => {
        const position = userLocationMarker.position;
        userPosition.lat = position.lat
        userPosition.lng = position.lng
        console.log(`ピンの座標：${userPosition.lat}, ${userPosition.lng}`)
    });
}

function handleLocationError() {
    const errorMessageElement = document.getElementById("errorMessage");
    errorMessageElement.innerHTML = `
        <p>位置情報が取得できませんでした。<br>
        所要時間と走行距離を表示させる場合は、ブラウザの設定から位置情報へのアクセスを許可してください。</p>`;
    errorMessageElement.style.display = "block";
}
</script>

<template>
    <div class="selectContainer">
        <select id="mySelect" v-model="selectedValue" @change="change">
            <option value="40">福岡県</option>
            <option value="41">佐賀県</option>
            <option value="42">長崎県</option>
            <option value="43">熊本県</option>
            <option value="44">大分県</option>
            <option value="45">宮崎県</option>
            <option value="46">鹿児島県</option>
            <option value="47">沖縄県</option>
        </select>
        <button class="button1" @click="search">search</button>
        <button class="button2" @click="displayUserLocation">現在地を表示</button>

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
    width: 220px;
    height: 40px;
    font-size: 16px;
}

.button3 {
    padding: 5px 10px;
    width: 140px;
    height: 40px;
    font-size: 16px;
    display: none;
}

.error-message {
    display: none;
    color: red;
    font-size: 14px;
    margin-bottom: 15px;
}
</style>
