<template>
  <v-app>
    <v-main>
      <notifications group="main" position="bottom right"/>
      <v-container class="application-container" fluid>
        <login v-if="apiKey == null"
               @apiKeyValidated="setApiKey"/>
        <notifications-hub v-if="apiKey != null" :apiKey="apiKey"/>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import NotificationsHub from './components/NotificationsHub.vue';
import Login from './components/Login.vue';

export default {
  name: 'App',
  components: {
    NotificationsHub,
    Login
  },
  methods: {
    setApiKey(key) {
      this.apiKey = key
    }
  },
  data: () => ({
    apiKey: null
  }),
};
</script>

<style scoped>

@import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro&display=swap');

.application-container {
  margin-top: 100px;
}


#app {
  font-family: 'Source Sans Pro', 'Avenir', Helvetica, Arial, sans-serif;
}

</style>
