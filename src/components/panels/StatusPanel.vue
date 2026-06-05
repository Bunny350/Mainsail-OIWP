<template>
    <div>
        <min-settings-panel />
        <klippy-state-panel />
        <panel
            v-if="klipperReadyForGui"
            :icon="statusIcon"
            :multipleIcons="true"
            :title="printerStateOutput"
            :collapsible="true"
            card-class="status-panel">
            <template #icon>
            <div v-if="['paused', 'printing'].includes(printer_state)" style="position: absolute; top: 6px; left: 16px">
                <v-progress-circular
                    :rotate="-90"
                    :size="30"
                    :width="5"
                    :value="printPercent"
                    :color="progressColor" />
                    
                    <Transition name="scale">
                    <div style="position: absolute; top: 0px; margin: 0px 0px 7px 7px" v-if="['paused'].includes(printer_state)">
                    <v-icon color="warning" small>{{ mdiPause }}</v-icon>
                    </div>
                    </transition>
            </div>
            </template>
            <template #buttons>
                <v-btn
                    v-for="button in filteredToolbarButtons"
                    :key="button.loadingName"
                    :color="button.color"
                    :loading="loadings.includes(button.loadingName)"
                    icon
                    tile
                    @click="button.click">
                    <v-tooltip top>
                        <template #activator="{ on, attrs }">
                            <v-icon v-bind="attrs" v-on="on">{{ button.icon }}</v-icon>
                        </template>
                        <span>{{ button.text }}</span>
                    </v-tooltip>
                </v-btn>
                <v-menu v-if="multiFunctionButton" left offset-y :close-on-content-click="false" class="pa-0">
                    <template #activator="{ on, attrs }">
                        <v-btn icon tile v-bind="attrs" v-on="on">
                            <v-icon>{{ mdiDotsVertical }}</v-icon>
                        </v-btn>
                    </template>
                    <v-list dense>
                        <v-list-item
                            v-for="(entry, index) in multiFunctionMenuButtonsFiltered"
                            :key="'multiFunction_' + index">
                            <v-btn small style="width: 100%" @click="entry.click()">
                                <v-icon left small>{{ entry.icon }}</v-icon>
                                {{ entry.text }}
                            </v-btn>
                        </v-list-item>
                    </v-list>
                </v-menu>
            </template>
            <status-panel-printstatus-thumbnail />
            <status-panel-exclude-object :show-dialog.sync="boolShowObjects" @update:showDialog="updateShowDialog" />
            <status-panel-pause-at-layer-dialog :show-dialog.sync="boolShowPauseAtLayer" />
            <template v-if="print_stats_message">
                <v-container>
                    <v-row>
                        <v-col class="py-2">
                            <span class="subtitle-2 px-0 text--disabled">
                                <v-icon class="mr-2 mt-1 float-left" color="warning" small>
                                    {{ mdiAlertOutline }}
                                </v-icon>
                                {{ print_stats_message }}
                            </span>
                        </v-col>
                    </v-row>
                </v-container>
                <v-divider class="mt-0 mb-0" />
            </template>
            <template v-if="display_message">
                <v-container>
                    <v-row class="flex-nowrap">
                        <v-col class="py-2" style="min-width: 0">
                            <span class="subtitle-2 px-0 text--disabled">
                                <v-icon class="mr-2 mt-1 float-left" small>{{ mdiMessageProcessingOutline }}</v-icon>
                                {{ display_message }}
                            </span>
                        </v-col>
                        <v-col class="col-auto py-2">
                            <v-icon class="text--disabled cursor-pointer" small @click="clearDisplayMessage">
                                {{ mdiCloseCircle }}
                            </v-icon>
                        </v-col>
                    </v-row>
                </v-container>
                <v-divider class="mt-0 mb-0" />
            </template>
            <v-tabs v-model="activeTab" fixed-tabs>
                <v-tab v-if="current_filename" href="#status">
                    <template v-if="legacyDynamicSpeedometer">
                        <v-icon v-if="currentSpeed < reach_mid_speed">{{ mdiSpeedometerSlow }} </v-icon>
                        <v-icon v-else-if="currentSpeed < max_velocity_one_and_quarter">{{ mdiSpeedometerMedium }}</v-icon>
                        <v-icon v-else>{{ mdiSpeedometer }}</v-icon>
                    </template>
                    <template v-else> <speedometer :style="'--var-transform: rotate(' + get_angle_by_speed + 'deg)'" /> </template>
                </v-tab>
                <v-tab v-if="displayFilesTab" href="#files">
                    <v-icon>{{ mdiFileDocumentMultipleOutline }}</v-icon>
                </v-tab>
                <v-tab v-if="displayHistoryTab" href="#history">
                    <v-icon>{{ mdiHistory }}</v-icon>
                </v-tab>
                <v-tab href="#jobqueue">
                    <v-badge :color="jobQueueBadgeColor" :content="jobsCount.toString()" :inline="true">
                    <template>
                        <template v-if="jobsCount == 0">
                            <v-icon color="disabled" >{{ mdiTray }}</v-icon>
                            </template> <template v-else-if="jobsCount == 1">
                            <v-icon color="disabled">{{ tray1l }}</v-icon>
                            </template> <template v-else-if="jobsCount == 2">
                            <v-icon color="disabled">{{ tray2l }}</v-icon>
                            </template> <template v-else-if="jobsCount > 2">
                            <v-icon color="disabled">{{ mdiTrayFull }}</v-icon>
                            </template>
                        </template>
                    </v-badge>
                </v-tab>
            </v-tabs>
            <v-divider class="my-0" />
            <v-tabs-items v-model="activeTab" class="_border-radius">
                <v-tab-item v-if="current_filename" value="status">
                    <status-panel-printstatus />
                </v-tab-item>
                <v-tab-item v-if="displayFilesTab" value="files">
                    <status-panel-gcodefiles />
                </v-tab-item>
                <v-tab-item v-if="displayHistoryTab" value="history">
                    <status-panel-history />
                </v-tab-item>
                <v-tab-item value="jobqueue">
                    <status-panel-jobqueue />
                </v-tab-item>
            </v-tabs-items>
        </panel>
        <confirmation-dialog
            v-model="showCancelJobDialog"
            :icon="mdiStopCircleOutline"
            :title="$t('CancelJobDialog.CancelJob')"
            :text="$t('CancelJobDialog.AreYouSure')"
            :action-button-text="$t('Buttons.Yes')"
            :cancel-button-text="$t('Buttons.No')"
            @action="cancelJob" />
    </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins, Watch } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import MinSettingsPanel from '@/components/panels/MinSettingsPanel.vue'
import KlippyStatePanel from '@/components/panels/KlippyStatePanel.vue'
import Speedometer from '@/components/ui/Speedometer.vue'
import StatusPanelPrintstatus from '@/components/panels/Status/Printstatus.vue'
import StatusPanelGcodefiles from '@/components/panels/Status/Gcodefiles.vue'
import StatusPanelHistory from '@/components/panels/Status/History.vue'
import StatusPanelJobqueue from '@/components/panels/Status/Jobqueue.vue'
import StatusPanelExcludeObject from '@/components/panels/Status/ExcludeObject.vue'
import StatusPanelPrintstatusThumbnail from '@/components/panels/Status/PrintstatusThumbnail.vue'
import StatusPanelPauseAtLayerDialog from '@/components/panels/Status/PauseAtLayerDialog.vue'
import Panel from '@/components/ui/Panel.vue'
import {
    mdiAlertOutline,
    mdiAlertCircleOutline,
    mdiBroom,
    mdiCheckBold,
    mdiCloseCircle,
    mdiCloseThick,
    mdiDotsVertical,
    mdiFileDocumentMultipleOutline,
    mdiHistory,
    mdiInformation,
    mdiLayersPlus,
    mdiMessageProcessingOutline,
    mdiPause,
    mdiPlay,
    mdiPrinter,
    mdiProgressQuestion,
    mdiSelectionRemove,
    mdiSpeedometer,
    mdiSpeedometerSlow,
    mdiSpeedometerMedium,
    mdiStop,
    mdiStopCircleOutline,
    mdiTray,
    mdiTrayFull,
} from '@mdi/js'
import { tray1l, tray2l } from '@/plugins/customIconsCommon'
import { PrinterStateMacro } from '@/store/printer/types'
import ConfirmationDialog from '@/components/dialogs/ConfirmationDialog.vue'

@Component({
    components: {
        ConfirmationDialog,
        KlippyStatePanel,
        MinSettingsPanel,
        Panel,
        'speedometer': Speedometer,
        StatusPanelExcludeObject,
        StatusPanelGcodefiles,
        StatusPanelHistory,
        StatusPanelJobqueue,
        StatusPanelPrintstatus,
        StatusPanelPrintstatusThumbnail,
        StatusPanelPauseAtLayerDialog,
    },
})
export default class StatusPanel extends Mixins(BaseMixin) {
    mdiAlertOutline = mdiAlertOutline
    mdiAlertCircleOutline = mdiAlertCircleOutline
    mdiCloseCircle = mdiCloseCircle
    mdiCheckBold = mdiCheckBold
    mdiCloseThick = mdiCloseThick
    mdiDotsVertical = mdiDotsVertical
    mdiFileDocumentMultipleOutline = mdiFileDocumentMultipleOutline
    mdiInformation = mdiInformation
    mdiHistory = mdiHistory
    mdiMessageProcessingOutline = mdiMessageProcessingOutline
    mdiPause = mdiPause
    mdiProgressQuestion = mdiProgressQuestion
    mdiSpeedometer = mdiSpeedometer
    mdiSpeedometerMedium = mdiSpeedometerMedium
    mdiSpeedometerSlow = mdiSpeedometerSlow
    mdiStopCircleOutline = mdiStopCircleOutline
    mdiTray = mdiTray
    tray1l = tray1l
    tray2l = tray2l
    mdiTrayFull = mdiTrayFull

    showCancelJobDialog = false
    boolShowObjects = false
    boolShowPauseAtLayer = false

    activeTab = 'files'
    lastFilename = ''

    get jobs() {
        return this.$store.getters['server/jobQueue/getJobs']
    }

    get jobsCount() {
        return this.$store.getters['server/jobQueue/getJobsCount']
    }

    get jobQueueBadgeColor() {
        return this.jobsCount > 0 ? 'primary darken-2' : 'grey darken-2'
    }

    get current_filename() {
        return this.$store.state.printer.print_stats?.filename ?? ''
    }

    get current_file() {
        return this.$store.state.printer.current_file ?? {}
    }

    get printPercent() {
        return Math.floor(this.$store.getters['printer/getPrintPercent'] * 100)
    }
    
    get progressColor() {
        if (['printing'].includes(this.printer_state)) return 'primary'
        else if (['paused'].includes(this.printer_state)) return 'warning'
        else if (['complete'].includes(this.printer_state)) return 'success'
        else if (['cancelled'].includes(this.printer_state)) return 'error'
    
    }
    
    get statusIcon() {
    if (this.printer_state !== '') {
        if (['complete'].includes(this.printer_state)) return mdiCheckBold
        else if (['cancelled'].includes(this.printer_state)) return mdiCloseThick
        else if (['error'].includes(this.printer_state)) return mdiAlertCircleOutline
        else return mdiInformation
        } else return mdiProgressQuestion
    
    }

    get statusIconPro() {
        return [
            {
            test1: ['complete'].includes(this.printer_state),
            icon: mdiCheckBold,
            }, {
            test1: ['cancelled'].includes(this.printer_state),
            icon: mdiCloseThick,
            }, {
            test1: ['error'].includes(this.printer_state),
            icon: mdiAlertCircleOutline,
            }, {
            icon: mdiInformation
            }
        ]
    }

    get printerStateOutput() {
        if (this.printer_state !== '') {
            const idle_timeout_state = this.$store.state.printer.idle_timeout?.state

            if (this.printer_state === 'standby' && idle_timeout_state === 'Printing') return 'Busy'

            if (this.printer_state !== '' && ['paused', 'printing'].includes(this.printer_state)) {
                return (
                    this.printPercent + '% ' + this.printer_state.charAt(0).toUpperCase() + this.printer_state.slice(1)
                )
            }

            return this.printer_state.charAt(0).toUpperCase() + this.printer_state.slice(1)
        }

        return this.$t('Panels.StatusPanel.Unknown')
    }

    get toolbarButtons() {
        return [
            {
                text: this.$t('Panels.StatusPanel.PausePrint'),
                color: 'warning',
                icon: mdiPause,
                loadingName: 'statusPrintPause',
                status: () => ['printing'].includes(this.printer_state),
                click: this.btnPauseJob,
            },
            {
                text: this.$t('Panels.StatusPanel.ResumePrint'),
                color: 'success',
                icon: mdiPlay,
                loadingName: 'statusPrintResume',
                status: () => ['paused'].includes(this.printer_state),
                click: this.btnResumeJob,
            },
            {
                text: this.$t('Panels.StatusPanel.CancelPrint'),
                color: 'error',
                icon: mdiStop,
                loadingName: 'statusPrintCancel',
                status: () => {
                    if (this.$store.state.gui.uiSettings.displayCancelPrint)
                        return ['paused', 'printing'].includes(this.printer_state)

                    return ['paused'].includes(this.printer_state)
                },
                click: this.btnCancelJob,
            },
            {
                text: this.$t('Panels.StatusPanel.ExcludeObject.ExcludeObject'),
                color: 'warning',
                icon: mdiSelectionRemove,
                loadingName: 'excludeObjectButton',
                status: () => {
                    if (this.multiFunctionButton || this.printing_objects.length < 2) return false

                    return ['paused', 'printing'].includes(this.printer_state)
                },
                click: this.btnExcludeObject,
            },
            {
                text: this.$t('Panels.StatusPanel.PauseAtLayer.PauseAtLayer'),
                color: 'warning',
                icon: mdiLayersPlus,
                loadingName: 'pauseAtLayer',
                status: () => {
                    if (this.multiFunctionButton || !this.displayPauseAtLayerButton) return false

                    return ['paused', 'printing'].includes(this.printer_state)
                },
                click: this.btnPauseAtLayer,
            },
            {
                text: this.$t('Panels.StatusPanel.ClearPrintStats'),
                color: 'primary',
                icon: mdiBroom,
                loadingName: 'statusPrintClear',
                status: () => ['error', 'complete', 'cancelled'].includes(this.printer_state),
                click: this.btnClearJob,
            },
            {
                text: this.$t('Panels.StatusPanel.ReprintJob'),
                color: 'primary',
                icon: mdiPrinter,
                loadingName: 'statusPrintReprint',
                status: () => ['error', 'complete', 'cancelled'].includes(this.printer_state),
                click: this.btnReprintJob,
            },
        ]
    }

    get filteredToolbarButtons() {
        return this.toolbarButtons.filter((button) => button.status())
    }

    get printing_objects() {
        return this.$store.state.printer.exclude_object?.objects ?? []
    }

    get display_message() {
        return this.$store.state.printer.display_status?.message ?? null
    }

    get print_stats_message() {
        return this.$store.state.printer.print_stats?.message ?? null
    }

    get layer_count() {
        return this.$store.state.printer.print_stats?.info?.total_layer ?? null
    }

    get multiFunctionMenuButtons() {
        return [
            {
                text: this.$t('Panels.StatusPanel.ExcludeObject.ExcludeObject'),
                loadingName: 'excludeObjectButton',
                icon: mdiSelectionRemove,
                status: () => this.printing_objects.length > 1,
                disabled: () => ['paused', 'printing'].includes(this.printer_state),
                click: this.btnExcludeObject,
            },
            {
                text: this.$t('Panels.StatusPanel.PauseAtLayer.PauseAtLayer'),
                loadingName: 'pauseAtLayer',
                icon: mdiLayersPlus,
                status: () => this.displayPauseAtLayerButton,
                disabled: () => ['paused', 'printing'].includes(this.printer_state),
                click: this.btnPauseAtLayer,
            },
        ]
    }

    get multiFunctionMenuButtonsFiltered() {
        return this.multiFunctionMenuButtons.filter((button) => button.status())
    }

    get multiFunctionButton() {
        if (!['paused', 'printing'].includes(this.printer_state)) return false

        return this.multiFunctionMenuButtonsFiltered.length > 1
    }

    get macros() {
        return this.$store.getters['printer/getMacros'] ?? []
    }

    get existsSetPauseAtLayer() {
        return this.macros.findIndex((macro: PrinterStateMacro) => macro.name === 'SET_PAUSE_AT_LAYER') !== -1
    }

    get existsSetPauseNextLayer() {
        return this.macros.findIndex((macro: PrinterStateMacro) => macro.name === 'SET_PAUSE_NEXT_LAYER') !== -1
    }

    get displayPauseAtLayerButton() {
        return this.layer_count !== null && (this.existsSetPauseAtLayer || this.existsSetPauseNextLayer)
    }

    get displayFilesTab() {
        const count = this.$store.state.gui.uiSettings.dashboardFilesLimit ?? 5

        return count > 0
    }

    get displayHistoryTab() {
        const count = this.$store.state.gui.uiSettings.dashboardHistoryLimit ?? 5

        return count > 0
    }

    get forceReducedMotion() {
        return this.$store.state.gui.uiSettings.forceReducedMotion ?? false
    }

    get legacyDynamicSpeedometer() {
        return this.$store.state.gui.uiSettings.legacyDynamicSpeedometer ?? false
    }
    
    get currentSpeed(): number {
        const requestedSpeed = this.$store.state.printer.gcode_move?.speed ?? 0
        const speedFactor = this.$store.state.printer.gcode_move?.speed_factor ?? 0
        const maxVelocity = this.$store.state.printer.toolhead?.max_velocity ?? 0
        const liveVelocity = Math.abs(this.$store.state.printer.motion_report?.live_velocity?.toFixed(0)) ?? null

        const speed = (requestedSpeed / 60) * speedFactor

        if (!isNaN(liveVelocity)) return liveVelocity
          else if (speed > maxVelocity) return maxVelocity
          else return speed.toFixed(0)
    }
    
    get reach_mid_speed(): number {
        if (this.$store.state.gui.uiSettings.legacyDynamicSpeedometer ?? false) {
            const max_velocity = this.$store.state.printer.toolhead?.max_velocity ?? 300
        
        if (max_velocity != 0 && Math.trunc(max_velocity / 3) < 80) return Math.trunc(max_velocity / 3)
            else return 80
            } else return 300
        }
    
    get max_velocity_one_and_quarter(): number {
        if (this.$store.state.gui.uiSettings.legacyDynamicSpeedometer ?? false) {
            const max_velocity = Math.trunc(this.$store.state.printer.toolhead?.max_velocity ?? 300)
            const max_velocity_one_and_quarter = Math.trunc(max_velocity / 1.5)
            return max_velocity_one_and_quarter
            } else return 150
        }
    
    get get_angle_by_speed(): number {
        if (!(this.$store.state.gui.uiSettings.legacyDynamicSpeedometer ?? false)) {
        const current_speed = this.currentSpeed
        const max_velocity = Math.trunc(this.$store.state.printer.toolhead?.max_velocity ?? 300)
        const min_angle = -170
        const max_angle = 80
        const angle_diff = 250
        var difference = 0
          
        if (!isNaN(current_speed)) difference = (Math.trunc(angle_diff / (Math.trunc(max_velocity * 0.8) / current_speed)) - 180)
        
        if (difference < min_angle) return min_angle
        else if (difference > max_angle) return max_angle
        else if (isNaN(difference) || (difference == null)) return 0
        else return difference
        } else return 0
    
    }

    mounted() {
        if (this.current_filename !== '') this.activeTab = 'status'
        if (!this.displayFilesTab) this.activeTab = 'history'
        if (!this.displayHistoryTab) this.activeTab = 'jobqueue'
    }

    @Watch('current_filename')
    current_filenameChanged(newVal: string) {
        if (newVal === '') this.activeTab = 'files'
        else if (this.lastFilename !== newVal) this.activeTab = 'status'

        this.lastFilename = newVal
    }

    clearDisplayMessage() {
        this.$socket.emit('printer.gcode.script', { script: 'M117' })
    }

    updateShowDialog(newVal: boolean) {
        this.boolShowObjects = newVal
    }

    btnPauseJob() {
        this.$socket.emit('printer.print.pause', {}, { loading: 'statusPrintPause' })
    }

    btnResumeJob() {
        this.$socket.emit('printer.print.resume', {}, { loading: 'statusPrintResume' })
    }

    btnExcludeObject() {
        this.boolShowObjects = true
    }

    btnPauseAtLayer() {
        this.boolShowPauseAtLayer = true
    }

    btnCancelJob() {
        const confirmOnCancelJob = this.$store.state.gui.uiSettings.confirmOnCancelJob
        if (confirmOnCancelJob) {
            this.showCancelJobDialog = true
            return
        }

        this.cancelJob()
    }

    cancelJob() {
        this.$socket.emit('printer.print.cancel', {}, { loading: 'statusPrintCancel' })
    }

    btnClearJob() {
        this.$socket.emit('printer.gcode.script', { script: 'SDCARD_RESET_FILE' }, { loading: 'statusPrintClear' })
    }

    btnReprintJob() {
        this.$socket.emit('printer.print.start', { filename: this.current_filename }, { loading: 'statusPrintReprint' })
    }
}
</script>

<style scoped>
._border-radius {
    border-bottom-left-radius: inherit;
    border-bottom-right-radius: inherit;
}

.theme--dark.v-tabs > .v-tabs-bar .v-tab:not(.v-tab--active) > .v-badge > .v-icon {
    color: rgba(255, 255, 255, 0.6);
}

svg.speedometer {

    --var-transform: rotate(-170deg)

}

</style>
