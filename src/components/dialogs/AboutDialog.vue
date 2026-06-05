<template>
    <v-tooltip right color="panel">
        <template #activator="{ on, attrs }">
            <v-icon v-bind="attrs" v-on="on">
                {{ mdiHelpCircleOutline }}
            </v-icon>
        </template>
        <v-container class="version-container pa-0">
            <div class="sub"><div class="logo"><img src="/img/logo.svg" alt="mainsail-logo" class="mainsail" /></div>
            <div class="versionNumber">v{{ mainsailVersion }}</div></div>
                        <div class="sub"><div class="logo"><v-icon small class="moonraker">{{ mdiMoonWaningCrescent }}</v-icon>
            </div>
            <div class="versionNumber">{{ moonrakerVersion }}</div></div>
                        <div class="sub"><div class="logo"><img src="/img/klipper.svg" class="klipper" alt="klipper-logo" /></div>
            <div class="versionNumber">{{ klipperVersion }}</div></div>
        </v-container>
    </v-tooltip>
</template>

<script lang="ts">
import BaseMixin from '../mixins/base'
import { Mixins } from 'vue-property-decorator'
import Component from 'vue-class-component'
import { mdiCloseThick, mdiHelpCircleOutline, mdiMoonWaningCrescent } from '@mdi/js'

@Component({})
export default class AboutDialog extends Mixins(BaseMixin) {
    mdiHelpCircleOutline = mdiHelpCircleOutline
    mdiCloseThick = mdiCloseThick
    mdiMoonWaningCrescent = mdiMoonWaningCrescent

    get mainsailVersion(): string {
        return this.$store.state.packageVersion
    }

    get klipperVersion(): string {
        return this.$store.state.printer?.software_version ?? ''
    }

    get moonrakerVersion(): string {
        return this.$store.state.server?.moonraker_version ?? ''
    }
}
</script>

<style scoped>

.logo {
    background-color: var(--v-btn-text-primary);
    width: 16px;
    height: 16px;
    margin-right: 6px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 5.5px

}

.versionNumber {
  display: flex;
  justify-content: left;
  align-items: center;
}

.logo img {
width: 12px;
height: 12px
}

.logo .klipper {
    /* transform: rotate(90deg); */
}
.logo .moonraker {
    transform: rotate(45deg);
    color: #ebc815;
}
.version-container {
    display: flex;
    flex-direction: column;
    justify-content: center
}
.version-container .sub {
    display: flex;
    align-items: center;
    height: 20px
}
</style>
