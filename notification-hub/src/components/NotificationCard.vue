<template>
    <v-col class="notification" align="center" justify="center" cols=6>
        <v-card :id="this.notification_id" dense>
            <v-expansion-panels v-model="panel" :disabled=disabled>
                <v-expansion-panel>
                    <v-expansion-panel-header>
                        <v-row dense>
                            <v-col cols=10>
                                <v-card-title>{{ this.notification.business_name }}</v-card-title>
                                <v-card-subtitle>
                                    {{ this.notification_id }}
                                </v-card-subtitle>
                            </v-col>
                            <v-col cols=1>
                                <v-tooltip bottom>
                                    <template v-slot:activator="{ on, attrs }">
                                        <v-icon class="tick-icon" @click="markAsRead"
                                                                  @click.stop
                                                                  v-bind="attrs"
                                                                  v-on="on">
                                            mdi-checkbox-marked-outline
                                        </v-icon>
                                    </template>
                                    <span>Mark as Read</span>
                                </v-tooltip>
                            </v-col>
                        </v-row>
                    </v-expansion-panel-header>
                    <v-expansion-panel-content>
                        <v-divider class="mx-4"></v-divider>
                        <v-row align="start" justify="start" dense>
                            <v-col align="start" justify="start">
                                <v-card-text>
                                    {{ this.notificationContent }}
                                </v-card-text>
                            </v-col>
                        </v-row>
                    </v-expansion-panel-content>
                </v-expansion-panel>
            </v-expansion-panels>
        </v-card>
    </v-col>
</template>

<script>

import axios from 'axios';

export default {
    name: 'NotificationCard',
    props: {
        notification: Object,
        notification_id: String,
        api_key: String
    },
    computed: {
        notificationContent() {
            let content = this.notification.notification
            return content.charAt(0).toUpperCase() + content.slice(1)
        },
        notificationColor() {
            const source = this.notification.metadata.source
            console.log(source)
            if (source != null) {
                return this.colorMappings[source]
            } else {
                return "red"
            }
        }
    },
    methods: {
        markAsRead() {
            // emit event used to re-fetch notifications
            let vm = this
            axios({
                method: 'patch',
                url: process.env.VUE_APP_BASE_API_URL + 'notifications/update/' + vm.notification_id,
                headers: {'X-ApiKey': vm.api_key}
            }).then(function (response) {
                console.log(response)
                vm.$notify({
                    group: 'main',
                    title: 'Notification Cleared',
                    type: 'success',
                    text: 'Successfully cleared notification ' + vm.notification_id
                })
                vm.$emit('notificationRead')
            }).catch(function (error) {
                console.log(error)
                vm.$notify({
                    group: 'main',
                    title: 'Failed to Clear',
                    type: 'error',
                    text: 'Failed to clear notification ' + vm.notification_id
                })
            })
        }
    },
    mounted() {
        let vm = this;
        var card = document.getElementById(vm.notification_id);
        card.style.borderLeft = vm.notificationColor
    },
    data: () => ({
        disabled: false,
        panel: [0, 1],
        colorMappings: {
            "web-scraper": "5px solid #6DCCFF",
            "yelp-api-connector": "5px solid #FF4B4B",
            "google-api-connector": "5px solid #C7FF4B"
        }
    })
}
</script>

<style scoped>


</style>