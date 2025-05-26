<script>

</script>

<template>
    <div class="resultContainer">
        <div class="list-title"><h3>選択した都道府県内の施設</h3></div>
        <div class="facility-list"></div>
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