<template>
    <div id="page">
        <div>适用于wsb/n - 
            <select v-model="selectedSchool">
                <option v-for="(_, key) in schools" :value="key">{{ key }}</option>
            </select>
        的地图筛选器</div>
        <component :is="schoolComponent" />
    </div>
</template>

<script setup>
import Wfust from './Wfust.vue'
import Sdut from './Sdut.vue'
import { computed, ref } from 'vue'
const schools = {
    '潍坊科技学院': Wfust,
    '山东理工大学': Sdut
}
const selectedSchool = ref(Object.keys(schools)[0])
const schoolComponent = computed(() => schools[selectedSchool.value])
</script>

<style scoped>
    h4{
        margin: 10px 0;
    }
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