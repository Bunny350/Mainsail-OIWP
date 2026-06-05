<template>
    <v-row :dense="dense">
        <v-col :class="firstColClasses">
            <v-row class="d-flex flex-row">
                <v-col v-if="loading" class="col-auto d-flex justify-center align-center pr-0">
                    <v-progress-circular indeterminate color="primary" :size="24" />
                </v-col>
                <v-col v-else-if="icon" class="col-auto d-flex justify-center align-center pr-0">
                    <v-icon>{{ icon }}</v-icon>
                </v-col>
                <v-col class="col d-flex justify-center flex-column">
                    <span class="settings-row-title">{{ title }}</span>
                    <span v-if="subTitle" class="settings-row-subtitle">{{ subTitle }}
                    <template v-if="hasTooltip">
                        <v-tooltip left max-width="500">
                        <template v-slot:activator="{ on, attrs }">
                          <v-icon
                            small
                            v-bind="attrs"
                            v-on="on"
                          >
                            {{ mdiInformationOutline }}
                          </v-icon>
                        </template>
                        <span>{{ tooltipDescription }} <v-icon>{{ tooltipIcon1 }}</v-icon> {{ tooltipAfterIcon1 }} <v-icon>{{ tooltipIcon2 }}</v-icon> {{ tooltipAfterIcon2 }} </span>
                        <template v-if="enabledExample && disabledExample">
                            <div style="display: flex; justify-content: space-around">
                                <div style="width: 150px; display: flex; flex-direction: column; justify-content: center">
                                <img :src="enabledExample" :alt="enabledExampleAlt" style="width: 150px; height: 150px; object-fit: contain"/>
                                <div style="display: flex; justify-content: center">
                                    <span v-if="(modeWatch)" class="example-selected">{{ $t('Settings.UiSettingsTab.Tooltip.enabled') }}</span>
                                    <span v-else>{{ $t('Settings.UiSettingsTab.Tooltip.enabled') }}</span>
                                </div>
                                </div>
                                <div style="width: 150px; display: flex; flex-direction: column; justify-content: center">
                                <img :src="disabledExample" :alt="disabledExampleAlt" style="width: 150px; height: 150px; object-fit: contain"/>
                                    <div style="display: flex; justify-content: center">
                                        <span v-if="(modeWatch)">{{ $t('Settings.UiSettingsTab.Tooltip.disabled') }}</span>
                                        <span v-else class="example-selected">{{ $t('Settings.UiSettingsTab.Tooltip.disabled') }}</span>
                                    </div>
                            </div>
                            </div>
                        </template>
                      </v-tooltip>
                    </template>
                    </span>
                </v-col>
            </v-row>
        </v-col>
        <v-col :class="secondColClasses">
            <slot />
        </v-col>
    </v-row>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from 'vue-property-decorator'
import BaseMixin from '../mixins/base'
import { TranslateResult } from 'vue-i18n'
import { mdiInformationOutline } from '@mdi/js'

@Component
export default class SettingsRow extends Mixins(BaseMixin) {
    mdiInformationOutline = mdiInformationOutline
    @Prop({ required: false, default: false }) readonly loading!: boolean
    @Prop({ required: false, default: '' }) readonly icon!: string
    @Prop({ required: true }) readonly title!: string | TranslateResult
    @Prop({ required: false }) readonly subTitle!: string | TranslateResult
    @Prop({ required: false }) readonly tooltipDescription!: string | TranslateResult
    @Prop({ required: false }) readonly tooltipAfterIcon1!: string | TranslateResult
    @Prop({ required: false }) readonly tooltipAfterIcon2!: string | TranslateResult
    @Prop({ required: false }) readonly tooltipIcon1!: string | TranslateResult
    @Prop({ required: false }) readonly tooltipIcon2!: string | TranslateResult
    @Prop({ required: false }) readonly enabledExampleAlt!: string | TranslateResult
    @Prop({ required: false }) readonly disabledExampleAlt!: string | TranslateResult
    @Prop({ required: false }) readonly enabledExample!: string
    @Prop({ required: false }) readonly disabledExample!: string
    @Prop({ required: false }) readonly modeWatch!: string
    @Prop({ required: false, default: false }) readonly dynamicSlotWidth!: boolean
    @Prop({ required: false, default: false }) readonly hasTooltip!: boolean
    @Prop({ required: false, default: false }) readonly mobileSecondRow!: boolean
    @Prop({ default: false }) readonly dense!: boolean

    get firstColClasses() {
        const dense = this.dense ? ' py-1' : ''
        const defaultClasses = ' d-flex justify-center' + dense

        if (this.dynamicSlotWidth) return 'col' + defaultClasses
        else if (this.mobileSecondRow) return 'col-12 col-md-6' + defaultClasses

        return 'col-6' + defaultClasses
    }

    get secondColClasses() {
        const dense = this.dense ? ' py-1' : ' settings-row-slot'
        const defaultClasses = ' d-flex justify-end align-center' + dense

        if (this.dynamicSlotWidth) return 'col-auto' + defaultClasses
        else if (this.mobileSecondRow) return 'col-12 col-md-6 pt-0 pt-md-3' + defaultClasses

        return 'col-6' + defaultClasses
    }
}
</script>

<style scoped>
.settings-row-title {
    display: block;
    width: 100%;
    font-weight: bold;
}

.settings-row-subtitle {
    display: block;
    font-size: 0.8em;
    line-height: 1.3;
    margin-top: 3px;
}

.settings-row-slot {
    min-height: 64px;
}

.example-selected {
    background-color: var(--v-secondary-base);
    padding: 0px 4px 0px 4px;
    border-radius: 8px;
    box-shadow: 0px 0px 3px #000
}

img {
    display: none
}
.menuable__content__active img, .fade-transition-leave-active img {
    display: initial
}

</style>
