<template>
    <v-row align="center" justify="center">
        <v-col align="center" justify="center" cols=8>
            <v-sheet elevation="3" color="white">
                <v-app-bar class="app-bar">
                    <v-app-bar-nav-icon></v-app-bar-nav-icon>
                    <v-toolbar-title>Texas Real Foods Notification Hub</v-toolbar-title>
                </v-app-bar>

                <v-container class="metadata-container" fluid>
                    <v-row align="center" justify="center">
                        <v-col align="center" justify="center" cols="4">
                            <v-row class="metric-value" align="center" justify="center" dense>
                                {{ this.notifications.length }}
                            </v-row>
                            <v-row class="metric-title" align="center" justify="center" dense>
                                Total Notifications
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
                url: process.env.VUE_APP_BASE_API_URL + '/api/notifications/unread',
                headers: {'X-ApiKey': vm.apiKey}
            }).then(function (response) {
                console.log(response)
                vm.notifications = response.data.notifications
            }).catch(function (error) {
                console.log(error)
            })
        }
    },
    data: () => ({
        notifications: []
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
    margin-top: 20px;
}

</style>