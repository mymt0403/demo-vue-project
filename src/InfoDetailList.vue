<script>

</script>

<template>
<div class="selectedFacilities">
    <h3>選択した都道府県内の施設</h3>
    <div id="facility-list"></div>
</div>
</template>

<style scoped>
.selectedFacilities {
    flex: 0 1 35%;
    height: 720px;
    width: 35%;
}

h3 {
    margin: 10px 0px
}

#facility-list {
    max-height: 640px;
    overflow-y: auto;
    padding: 10px;
}
</style>