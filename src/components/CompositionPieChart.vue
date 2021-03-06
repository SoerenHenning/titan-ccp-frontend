<template>
    <div class="card">
        <div class="card-body">
            <h5 class="card-title">Composition</h5>
            <loading-spinner :is-loading="isLoading" :is-error="isError">
                <div class="c3-container"></div>
            </loading-spinner>
        </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component, Prop, Watch } from "vue-property-decorator"
import { SensorRegistryRequester, AggregatedSensor, Sensor, SensorRegistry } from '../SensorRegistry'
import { HTTP } from "../http-common"
import LoadingSpinner from "./LoadingSpinner.vue"
import { ChartAPI, generate } from 'c3'
import 'c3/c3.css'

@Component({
    components: {
        LoadingSpinner
    }
})
export default class CompositionPieChart extends Vue {

    @Prop({ required: true }) sensor!: AggregatedSensor

    private isLoading = false
    private isError = false

    private chart!: ChartAPI

    mounted() {
        this.chart = generate({
            bindto: this.$el.querySelector(".c3-container") as HTMLElement,
            data: {
                columns: [],
                type : 'pie'
            },
            tooltip: {
                show: false
            }
        })
        this.updateChart()
    }

    @Watch('sensor')
    onSensorChanged() {
        this.updateChart()
    }

    private updateChart() {
        this.isLoading = true
        Promise.all(this.sensor.children.map(child => {
            let resource = child instanceof AggregatedSensor ? 'aggregated-power-consumption' : 'power-consumption'
            return HTTP.get(resource + '/' + child.identifier + '/latest')
                .then(response => {
                    // JSON responses are automatically parsed.
                    let value
                    if (response.data.length <= 0) {
                        value = 0
                    } else if (child instanceof AggregatedSensor) {
                        value = response.data[0].sumInW
                    } else {
                        value = response.data[0].valueInW
                    }
                    return <[string, number]> [child.title, value]
                })
        })).catch(e => {
            console.error(e)
            this.isError = true
            return []
        }).then(columns => {
            this.isLoading = false
            this.chart.load({
                columns: columns,
                unload: true
            })
        })
    }

}
</script>

<style scoped>
    .c3-container {
        height: 300px;
    }
</style>
