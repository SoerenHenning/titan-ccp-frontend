<template>
    <b-breadcrumb class="sensor-parents">
        <b-breadcrumb-item v-for="sensor in parents" :key="sensor.identifier" @click="select(sensor)" href="#" :text="sensor.title" />
        <b-breadcrumb-item active :text="sensor.title" />
    </b-breadcrumb>
</template>

<script lang="ts">
import { Vue, Component, Prop, Emit } from "vue-property-decorator"
import { Sensor, AggregatedSensor } from '../SensorRegistry'

import BootstrapVue from 'bootstrap-vue'
import 'bootstrap/dist/css/bootstrap.css'//TODO necessary?
import 'bootstrap-vue/dist/bootstrap-vue.css' //TODO necessary?

@Component
export default class SensorParents extends Vue {

    @Prop({ required: true }) sensor!: Sensor

    get parents () {
        let parents = [this.sensor]
        while (parents[parents.length - 1].parent) {
            parents.push(parents[parents.length - 1].parent as AggregatedSensor) // cast shouldn't be necessary
        }
        return parents.reverse().slice(0, -1)
    }

    @Emit()
    select(sensor: AggregatedSensor) {
    }

}
</script>

<style scoped>
    .sensor-parents {
        margin-bottom: 0;
    }
</style>