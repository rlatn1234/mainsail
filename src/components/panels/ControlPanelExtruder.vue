<style lang="scss" scoped>
.btnHomeAxis {
    width: 36px;
    min-width: 36px !important;
}

.btnMinWidthAuto {
    min-width: auto !important;
}

.steps {
    width: 100%;
    > div {
        width: 100%;
        display: flex;
        > button {
            flex-grow: 1;
        }
    }
}
</style>

<template>
    <div class="mt-6">
        <v-row>
            <v-col class="pa-0">
                <v-divider></v-divider>
            </v-col>
        </v-row>
        <v-row class="">
            <v-col class="col col-md-6 pt-2">
                <span class="text--disabled" style="font-size: .9em">{{ $t("Panels.ControlPanel.FeedAmountIn") }} [mm]</span>
                <v-btn-toggle class="mt-1" dense no-gutters style="flex-wrap: nowrap; width: 100%;" >
                    <v-btn v-for="amount in feedamountsSorted" v-bind:key="amount" @click="setFeedAmount(amount)" dense :class="(amount === currentFeedAmount ? 'v-btn--active' : '') + ' btnMinWidthAuto flex-grow-1 px-0 _btnFeedrate'">{{ amount }}</v-btn>
                </v-btn-toggle>
            </v-col>
            <v-col class="col col-md-6 pt-2">
                <span class="text--disabled" style="font-size: .9em">{{ $t("Panels.ControlPanel.FeedrateIn") }} [mm/s]</span>
                <v-btn-toggle class="mt-1" dense no-gutters style="flex-wrap: nowrap; width: 100%;" >
                    <v-btn v-for="rate in feedratesSorted" v-bind:key="rate" @click="setFeedrate(rate)" dense :class="(rate === currentFeedRate ? 'v-btn--active' : '') + ' btnMinWidthAuto flex-grow-1 px-0 _btnFeedrate'">{{ rate }}</v-btn>
                </v-btn-toggle>
            </v-col>
        </v-row>
        <v-row class="">
            <v-col class="col text-center pt-0">
                <v-btn small @click="sendRetract()" class="mx-3" :loading="loadings.includes('btnRetract')" :disabled="!boolExtrudePossible"><v-icon small class="mr-1">mdi-arrow-up-bold</v-icon> {{ $t('Panels.ControlPanel.Retract') }}</v-btn>
                <v-btn small @click="sendDetract()" class="mx-3" :loading="loadings.includes('btnDetract')" :disabled="!boolExtrudePossible"><v-icon small class="mr-1">mdi-arrow-down-bold</v-icon> {{ $t('Panels.ControlPanel.Extrude') }}</v-btn>
            </v-col>
        </v-row>
    </div>
</template>

<script lang="ts">
import {Component, Mixins} from 'vue-property-decorator'
import BaseMixin from '../mixins/base'

@Component
export default class ControlPanelExtruder extends Mixins(BaseMixin) {

    get feedamounts() {
        return this.$store.state.gui.dashboard?.extruder?.feedamounts ?? []
    }

    get feedrates() {
        return this.$store.state.gui.dashboard?.extruder?.feedrates ?? []
    }

    get feedamountsSorted() {
        return [...this.feedamounts].sort((a,b) => { return b-a })
    }

    get feedratesSorted () {
        return [...this.feedrates].sort((a,b) => { return b-a })
    }

    get currentFeedAmount() {
        return parseFloat(this.$store.state.gui.dashboard.extruder.feedamount)
    }

    set currentFeedAmount(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: 'dashboard.extruder.feedamount', value: newVal })
    }

    get currentFeedRate() {
        return parseFloat(this.$store.state.gui.dashboard.extruder.feedrate)
    }

    set currentFeedRate(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: 'dashboard.extruder.feedrate', value: newVal })
    }

    get boolExtrudePossible() {
        return this.$store.getters['printer/getExtrudePossible']
    }

    doSend(gcode: string) {
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode })
    }

    setFeedAmount(value: number) {
        this.currentFeedAmount = value
    }

    setFeedrate(value: number) {
        this.currentFeedRate = value
    }

    sendRetract() {
        const gcode = 'M83\nG1 E-'+this.currentFeedAmount+' F'+(this.currentFeedRate * 60)
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode }, { loading: 'btnRetract' })
    }

    sendDetract() {
        const gcode = 'M83\nG1 E'+this.currentFeedAmount+' F'+(this.currentFeedRate * 60)
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode }, { loading: 'btnDetract' })
    }
}
</script>
