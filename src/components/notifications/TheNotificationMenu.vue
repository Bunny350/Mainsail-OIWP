<template>
    <v-menu
        v-model="boolMenu"
        bottom
        :left="!isMobile"
        offset-y
        :close-on-click="true"
        :close-on-content-click="false"
        origin="center center"
                transition="oiwp-transition"
                content-class="notifications"
        :min-width="isMobile ? '100%' : null">
        <template #activator="{ on: menu, attrs }">
        <v-tooltip :disabled="boolMenu" bottom :open-on-click="false" :open-on-focus="false" z-index="255">
            <template v-slot:activator="{ on: tooltip }">
                <v-btn icon tile class="minwidth-0" v-bind="attrs" v-on="attrs['aria-expanded'] === 'false' ? {...tooltip, ...menu} : { ...menu }">
                    <v-badge
                        :content="notifications.length"
                        :value="notifications.length > 0"
                        :color="colorBadge"
                        overlap>
                        <v-icon>{{ attrs['aria-expanded'] === 'false' ? mdiBellOutline : mdiBell }}</v-icon>
                    </v-badge>
                </v-btn>
            </template>
            <span>
            <div><template v-if="notifications.length > 0"> {{ $tc('App.TopBar.NewNotification', notifications.length, { number: notifications.length } ) }} </template> <template v-else> {{ $t('App.Notifications.NoNotification') }} </template> </div>
            <div> <template v-if="existsHighAnnouncements" >  {{ $t('App.TopBar.NotificationImportant', { number: getHighAnnouncements.length })}} </template> </div>
            <div> <template v-if="existsCriticalAnnouncements" >  {{ $t('App.TopBar.NotificationCritical', { number: getCriticalAnnouncements.length })}} </template> </div>
            </span>
            </v-tooltip>
        </template>
        <v-card flat :min-width="300" :max-width="isMobile ? null : 400" class="notifications inner">
            <template v-if="notifications.length">
                <overlay-scrollbars class="announcement-menu__scrollbar">
                    <v-card-text>
                        <template v-for="(entry, index) in notifications">
                            <notification-menu-entry
                                :key="entry.id"
                                :entry="entry"
                                :class="index < notifications.length - 1 ? '' : 'mb-0'"
                                :parent-state="boolMenu" />
                        </template>
                    </v-card-text>
                </overlay-scrollbars>
                <template v-if="notifications.length > 1">
                    <v-divider />
                    <v-card-actions>
                        <v-spacer />
                        <v-btn text color="primary" class="mr-2" @click="dismissAll">
                            <v-icon left>{{ mdiCloseBoxMultipleOutline }}</v-icon>
                            {{ $t('App.Notifications.DismissAll') }}
                        </v-btn>
                    </v-card-actions>
                </template>
            </template>
            <v-card-text v-else class="text-center">
                <span class="text-disabled">{{ $t('App.Notifications.NoNotification') }}</span>
            </v-card-text>
        </v-card>
    </v-menu>
</template>

<script lang="ts">
import BaseMixin from '@/components/mixins/base'
import { Component, Mixins } from 'vue-property-decorator'
import NotificationMenuEntry from '@/components/notifications/NotificationMenuEntry.vue'
import { mdiBell, mdiBellOutline, mdiCloseBoxMultipleOutline } from '@mdi/js'
import { GuiNotificationStateEntry } from '@/store/gui/notifications/types'

@Component({
    components: { NotificationMenuEntry },
})
export default class TheNotificationMenu extends Mixins(BaseMixin) {
    mdiBell = mdiBell
    mdiBellOutline = mdiBellOutline
    mdiCloseBoxMultipleOutline = mdiCloseBoxMultipleOutline

    boolMenu = false

    get notifications() {
        return this.$store.getters['gui/notifications/getNotifications'] ?? []
    }
    
    get getCriticalAnnouncements() {
        return this.notifications.filter((entry: GuiNotificationStateEntry) => entry.priority === 'high') ?? []
    }
    
    get getHighAnnouncements() {
        return this.notifications.filter((entry: GuiNotificationStateEntry) => entry.priority === 'high') ?? []
    }

    get existsCriticalAnnouncements() {
        return this.notifications.filter((entry: GuiNotificationStateEntry) => entry.priority === 'critical').length > 0
    }

    get existsHighAnnouncements() {
        return this.notifications.filter((entry: GuiNotificationStateEntry) => entry.priority === 'high').length > 0
    }

    get countNormalAnnouncements() {
        return this.notifications.filter((entry: GuiNotificationStateEntry) => entry.priority === 'normal').length
    }

    get colorBadge() {
        if (this.existsCriticalAnnouncements) return 'error'
        if (this.existsHighAnnouncements) return 'warning'

        return 'primary'
    }

    dismissAll() {
        this.notifications.forEach(async (entry: GuiNotificationStateEntry) => {
            if (entry.id.startsWith('announcement')) {
                await this.$store.dispatch('gui/notifications/close', { id: entry.id })
            }

            await this.$store.dispatch('gui/notifications/dismiss', { id: entry.id, type: 'reboot', time: null })
        })
    }
}
</script>

<style scoped>
.announcement-menu__scrollbar {
    max-height: 500px;
}

    .notifications.oiwp-transition-enter-active, .notifications.oiwp-transition-leave-active {
    right: 118px
}
    .notifications.oiwp-transition-enter, .notifications.oiwp-transition-leave-to {
    left: calc(100% - 270px + 118px)!important;
}
    .notifications.oiwp-transition-enter-to, .notifications.oiwp-transition-leave {
    
}
</style>
