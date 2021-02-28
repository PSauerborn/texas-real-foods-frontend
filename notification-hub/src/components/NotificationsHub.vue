<template>
    <v-row align="center" justify="center">
        <v-col align="center" justify="center" cols=8>
            <v-sheet elevation="3" color="white">
                <v-app-bar class="app-bar">
                    <v-app-bar-nav-icon @click="drawer = true"></v-app-bar-nav-icon>
                    <v-toolbar-title>Texas Real Foods Notification Hub</v-toolbar-title>
                </v-app-bar>

                <v-navigation-drawer v-model="drawer" width="250" absolute temporary>
                    <v-list nav dense>
                        <v-list-item-group v-model="group">
                            <v-list-item @click="getNotifications">
                                <v-list-item-icon>
                                    <v-icon>mdi-all-inclusive</v-icon>
                                </v-list-item-icon>
                                <v-list-item-title>All Notifications</v-list-item-title>
                            </v-list-item>
                            <v-list-item @click="getNotificationsBySource('yelp-api-connector')">
                                <v-list-item-icon>
                                    <v-icon>mdi-handshake</v-icon>
                                </v-list-item-icon>
                                <v-list-item-title>Yelp Notifications</v-list-item-title>
                            </v-list-item>
                            <v-list-item @click="getNotificationsBySource('google-api-connector')">
                                <v-list-item-icon>
                                    <v-icon>mdi-google</v-icon>
                                </v-list-item-icon>
                                <v-list-item-title>Google Notifications</v-list-item-title>
                            </v-list-item>
                            <v-list-item @click="getNotificationsBySource('web-scraper')">
                                <v-list-item-icon>
                                    <v-icon>mdi-web</v-icon>
                                </v-list-item-icon>
                                <v-list-item-title>Web Notifications</v-list-item-title>
                            </v-list-item>
                            <v-list-item @click="getNotificationsBySource('parked-domain-checker')">
                                <v-list-item-icon>
                                    <v-icon>mdi-web</v-icon>
                                </v-list-item-icon>
                                <v-list-item-title>Parked Domain Notifications</v-list-item-title>
                            </v-list-item>
                            <v-list-item @click="markAllAsRead">
                                <v-list-item-icon>
                                    <v-icon>mdi-delete</v-icon>
                                </v-list-item-icon>
                                <v-list-item-title>Clear All Notifications</v-list-item-title>
                            </v-list-item>
                        </v-list-item-group>
                    </v-list>
                </v-navigation-drawer>

                <v-container class="metadata-container" fluid>
                    <v-row align="center" justify="center">
                        <v-col align="center" justify="center" cols="4">
                            <v-row class="metric-value" align="center" justify="center" dense>
                                {{ this.totalNotificationsCount }}
                            </v-row>
                            <v-row class="metric-title" align="center" justify="center" dense>
                                Total Notifications
                            </v-row>
                        </v-col>
                        <v-col align="center" justify="center" cols="4">
                            <v-row class="metric-value" align="center" justify="center" dense>
                                {{ this.notifications.length }}
                            </v-row>
                            <v-row class="metric-title" align="center" justify="center" dense>
                                Displayed Notifications
                            </v-row>
                        </v-col>
                        <v-col align="center" justify="center" cols="4">
                            <v-row class="metric-value" align="center" justify="center" dense>
                                {{ this.businessCount }}
                            </v-row>
                            <v-row class="metric-title" align="center" justify="center" dense>
                                Unique Businesses
                            </v-row>
                        </v-col>
                    </v-row>
                </v-container>
                <v-divider class="mx-4"></v-divider>
                <v-container class="notifications-container" fluid>
                    <v-row align="center" justify="center" dense>
                        <notification-card v-for="(notification, i) in notifications"
                            @notificationRead="getNotifications"
                            :notification_id=notification.notification_id
                            :notification=notification.notification
                            :api_key=apiKey
                            :key="i"/>
                    </v-row>
                </v-container>
            </v-sheet>
        </v-col>
    </v-row>
</template>

<script>

import NotificationCard from './NotificationCard.vue'
import axios from 'axios';

export default {
    name: 'NotificationsHub',
    components: {
        NotificationCard
    },
    props: {
        apiKey: String
    },
    computed: {
        businessCount() {
            const businesses = new Set()
            this.notifications.forEach(element => {
                businesses.add(element.notification.business_name)
            })
            return businesses.size
        }
    },
    methods: {
        getNotifications() {
            let vm = this
            axios({
                method: 'get',
                url: process.env.VUE_APP_BASE_API_URL + 'notifications/unread',
                headers: {'X-ApiKey': vm.apiKey}
            }).then(function (response) {
                console.log(response)
                vm.notifications = response.data.notifications
                vm.totalNotificationsCount = response.data.count
                vm.$notify({
                    group: 'main',
                    title: 'Notification(s) Retrieved',
                    type: 'success',
                    text: 'Successfully retrieve notifications'
                })
            }).catch(function (error) {
                console.log(error)
                vm.$notify({
                    group: 'main',
                    title: 'Failed to retrieve Notifications',
                    type: 'error',
                    text: 'Failed to retrieve Notifications'
                })
            })
        },
        getNotificationsBySource(source) {
            let vm = this
            axios({
                method: 'get',
                url: process.env.VUE_APP_BASE_API_URL + 'notifications/unread?filter=source:' + source,
                headers: {'X-ApiKey': vm.apiKey}
            }).then(function (response) {
                console.log(response)
                vm.notifications = response.data.notifications
                vm.totalNotificationsCount = response.data.count
                vm.$notify({
                    group: 'main',
                    title: 'Notification(s) Retrieved',
                    type: 'success',
                    text: 'Successfully retrieve notifications for source ' + source
                })
            }).catch(function (error) {
                console.log(error)
                vm.$notify({
                    group: 'main',
                    title: 'Failed to retrieve Notifications',
                    type: 'error',
                    text: 'Failed to retrieve Notifications for source' + source
                })
            })
        },
        markAllAsRead() {
            let notifications = []
            this.notifications.forEach(ele => {
                notifications.push(ele.notification_id)
            })

            let vm = this
            axios({
                method: 'patch',
                url: process.env.VUE_APP_BASE_API_URL + 'notifications/update-batch',
                data: {'notifications': notifications},
                headers: {'X-ApiKey': vm.apiKey}
            }).then(function (response) {
                console.log(response)
                vm.$notify({
                    group: 'main',
                    title: 'Notification(s) Cleared',
                    type: 'success',
                    text: 'Successfully cleared all notifications'
                })
                vm.getNotifications()
            }).catch(function (error) {
                console.log(error)
                vm.$notify({
                    group: 'main',
                    title: 'Failed to Clear',
                    type: 'error',
                    text: 'Failed to batch clear notifications'
                })
            })
        }
    },
    data: () => ({
        notifications: [],
        totalNotificationsCount: 0,
        drawer: false,
        group: null
    }),
    mounted() {
        this.getNotifications();
    }
}
</script>

<style scoped>

.app-bar {
    margin-bottom: 20px;
}

.metric-title {
    font-weight: bold;
}

.metric-value {
    font-size: 50px;
    font-weight: bold;
}

.metadata-container {
    margin-bottom: 20px;
}

.notifications-container {

}

</style>