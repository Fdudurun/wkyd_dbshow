<template>
    <div id="page">
        <h4>适用于wsb/n的地图筛选器</h4>
        <MapsList />
    </div>
</template>

<script setup>
import MapsList from './MapsList.vue';
</script>

<style scoped>
    #page {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100%;
        width: 100%;
        padding: 0;
        margin: 0;
    }
</style>