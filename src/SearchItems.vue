<script>
function fetchData() {
    /** mySelectの項目(value)を取得する */
    const selectBox = document.getElementById('mySelect');
    const selectedValue = selectBox.value;

    fetch(`/api/center/\${selectedValue}`)
        .then(response => response.json())
        .then(data =>
            map = new google.maps.Map(document.getElementById("map"), {
                center: {
                    lat: data.latitude,
                    lng: data.longitude,
                },
                zoom: 9,
                mapId: "4504f8b37365c3d0",
            })
        );

    fetch(`/api/data/\${selectedValue}`)
        .then(response => response.json())
        .then(data => putPins(data));
}

async function putPins(pins) {
    const { AdvancedMarkerElement } = await google.maps.importLibrary("marker");
    const { PinElement } = google.maps.importLibrary("marker");
    pins.forEach((pin, i) => {
        new AdvancedMarkerElement({
            map: map,
            position: {
                lat: pin.latitude,
                lng: pin.longitude,
            },
            title: pin.facilityName,
            content: new PinElement({
                glyph: `\${i + 1}`,
                glyphColor: "white",
                scale: 1,
            }).element,
        });
    });
    displayFacilities(pins);
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

                const faPin = new PinElement({
                    glyph: icon,
                    glyphColor: "#000000",
                    background: "#FF6633",
                    borderColor: "#FF0000",
                });

                /** 現在地をマークする */
                const geolocationMarker = new AdvancedMarkerElement({
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

function handleLocationError() {
    const errorMessageElement = document.getElementById("errorMessage");
    errorMessageElement.innerHTML = `
        <p>位置情報が取得できませんでした。<br>
        位置情報を表示させる場合はブラウザの設定から位置情報へのアクセスを許可してください。</p>`;
    errorMessageElement.style.display = "inline";
}
</script>

<template>
    <div class="container">
        <select id="mySelect">
            <option value="40">福岡県</option>
            <option value="41">佐賀県</option>
            <option value="42">長崎県</option>
            <option value="43">熊本県</option>
            <option value="44">大分県</option>
            <option value="45">宮崎県</option>
            <option value="46">鹿児島県</option>
            <option value="47">沖縄県</option>
        </select>
        <button class="button1" @click="fetchData">search</button>
        <button class="button2" @click="showUserLocation">display your location</button>
    </div>
    <div class="error" style="vertical-align: top;">
        <span id="errorMessage" class="error-message"></span>
    </div>
</template>

<style>
.container {
    gap: 10px;
    width: 100%;
    max-width: 960px;
    display: flex;
    align-items: left;
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
}
</style>
