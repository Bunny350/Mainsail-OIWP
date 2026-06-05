<template>
    <div>
        <v-tooltip right :open-delay="500" :disabled="navigationStyle !== 'iconsOnly'">
            <template #activator="{ on, attrs }">
                <v-list-item
                    :router="to !== undefined"
                    :to="to"
                    :href="href"
                    :target="target"
                    :class="itemClass"
                    v-bind="attrs"
                    v-on="on">
                    <v-list-item-icon class="my-3 mr-3 menu-item-icon">
                        <Transition name="scale" mode="out-in">
                        <div style="width: 24px; height: 24px; position: absolute; top: 10px" v-if="item.to === '/' && !isActive && ['paused', 'printing'].includes(printer_state)">
                            <v-progress-circular
                                :rotate="-90"
                                :size="24"
                                :width="4"
                                :value="printPercent"
                                :color="progressColor" />
                            <Transition name="scale">
                                <div style="position: absolute; top: 0px; margin: 0px 0px 4px 4px!important" v-if="['paused'].includes(printer_state)">
                                    <v-icon color="warning" small>{{ mdiPause }}</v-icon>
                                </div>
                            </transition>
                        </div>
                                <span v-else-if="item.to === '/' && !isActive && ['complete'].includes(printer_state)" style="display: inline-flex" >
                                <v-icon>{{ mdiCheckBold }}</v-icon>
                                </span>
                                <v-icon v-else>{{ icon }}</v-icon>
                        </Transition>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title tile class="menu-item-title">
                            {{ title }}
                        </v-list-item-title>
                    </v-list-item-content>
                </v-list-item>
            </template>
            <span>{{ title }}</span>
        </v-tooltip>
        <v-divider v-if="borderBottom" class="my-1" />
    </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins, Prop } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import { NaviPoint } from '@/components/mixins/navigation'
import { mdiCheckBold, mdiPause } from '@mdi/js'

@Component
export default class SidebarItem extends Mixins(BaseMixin) {
    mdiCheckBold = mdiCheckBold
    mdiPause = mdiPause

    @Prop({ type: Object, required: true }) item!: NaviPoint

    get navigationStyle() {
        return this.$store.state.gui.uiSettings.navigationStyle
    }

    get icon() {
        return this.item.icon
    }

    get title() {
        return this.item.title
    }

    get to() {
        return this.item.to ?? undefined
    }

    get href() {
        return this.item.href ?? undefined
    }

    get target() {
        return this.item.target ?? undefined
    }

    get borderBottom() {
        return this.item.to === '/allPrinters'
    }

    get isActive(): boolean {
        if (this.item.target === '_blank' || !this.item.to) return false

        return this.$route.path === this.item.to
    }

    get itemClass() {
        return {
            'small-list-item': true,
            'active-nav-item': this.isActive,
        }
    }

    get progressColor() {
        if (['printing'].includes(this.printer_state)) return 'primary'
        else if (['paused'].includes(this.printer_state)) return 'warning'
    }
    
    get printPercent() {
        return Math.floor(this.$store.getters['printer/getPrintPercent'] * 100)
    }

}
</script>

<style scoped>
.small-list-item {
    height: var(--sidebar-menu-item-height);
}

.active-nav-item {
    border-right: 4px solid var(--v-primary-base);
}

.menu-item-icon {
    opacity: 0.85;
}

.menu-item-title {
    line-height: 30px;
    font-size: 14px;
    font-weight: 600;

}
</style>
