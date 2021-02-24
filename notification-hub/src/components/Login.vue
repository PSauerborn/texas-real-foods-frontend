<template>
    <v-row align="center" justify="center">
        <v-col align="center" justify="center" cols="6">
            <v-sheet elevation="2">
                <v-row align="center" justify="center">
                    <v-col align="center" justify="center">
                        <h1>Texas Real Foods Notification Hub</h1>
                    </v-col>
                </v-row>
                <v-row align="center" justify="center">
                    <v-col align="center" justify="center" cols="6">
                        <v-text-field
                            v-model="apiKey"
                            label="Enter API Key"
                            clearable>
                        </v-text-field>
                        <v-btn color="primary" @click="validateApiKey">
                            Submit
                        </v-btn>
                    </v-col>
                </v-row>
            </v-sheet>
        </v-col>
    </v-row>
</template>

<script>

import axios from 'axios';

export default {
    name: 'Login',
    methods: {
        validateApiKey() {
            let vm = this
            axios({
                method: 'get',
                url: process.env.VUE_APP_BASE_API_URL + '/authenticate',
                headers: {'X-ApiKey': vm.apiKey}
            }).then(function (response) {
                console.log(response)
                vm.$emit('apiKeyValidated', vm.apiKey)
            }).catch(function (error) {
                console.log(error)
                if (error.response.status == 500) {
                    vm.$notify({
                    group: 'main',
                    title: 'authentication',
                    type: 'error',
                    text: 'Internal server error'
                })
                } else {
                    vm.$notify({
                    group: 'main',
                    title: 'authentication',
                    type: 'error',
                    text: 'Invalid API Key'
                })
                }
            })
        }
    },
    data: () => ({
        apiKey: ''
    })
}
</script>

<style scoped>

</style>