<template>
  <v-app>
    <v-app-bar color="primary" dark app clipped-left>
      <v-toolbar-title>
        <router-link to="/" class="toolbar__top v-toolbar__content"
          >NEM Authn Prototype</router-link
        >
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text to="/">Home</v-btn>
      <v-btn text v-if="!$auth.isAuthenticated" @click="login">Log in</v-btn>
      <v-btn text v-if="$auth.isAuthenticated" @click="logout">Log out</v-btn>
      <v-btn text to="/signup">SignUp</v-btn>
      <v-btn text href="https://t-miya19890131.medium.com/nem-authn-%E6%9A%97%E5%8F%B7%E8%B3%87%E7%94%A3%E3%81%AE%E6%89%80%E6%9C%89%E6%A8%A9%E3%81%A7%E8%AA%8D%E8%A8%BC%E3%81%99%E3%82%8Bidp%E3%81%AE%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E3%83%97%E3%83%AD%E3%83%88%E3%82%BF%E3%82%A4%E3%83%97-1819ea9e08d2" target="_blank">About</v-btn>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <loading v-show="loading" />
        <router-view v-show="!loading" v-bind:data="oaMosaics" />

      </v-container>
      <v-footer app>
        <div style="margin: 0 auto">ツイベガ君 & みやとも</div>
      </v-footer>
    </v-main>
  </v-app>
</template>

<script>
import Loading from "./components/container/Loading.vue";

export default {
  data() {
    return {
      drawer: null,
      loading: false,
      oaMosaics: [],
    };
  },

  watch: {
    "$auth.user": {
      immediate: true,
      handler: async function (val) {
        if (val && this.$auth.isAuthenticated) {
          this.loading = true;
          const gotMosaics = [];
          const promises = [];
          try {
            const namespaces = await this.$nem.getOwnedMosaics(
              this.$auth.user.nickname
            );
            console.log(namespaces);
            for (const ns of namespaces) {
              if (!gotMosaics.includes(ns.mosaicId.namespaceId)) {
                gotMosaics.push(ns.mosaicId.namespaceId);
                promises.push(
                  this.$nem.getMosaicDetail(ns.mosaicId.namespaceId)
                );
              }
            }

            let mosaics = await Promise.all(promises);
            mosaics = mosaics
              .flat(2)
              .filter((i) => i.imageUrl && i.imageUrl !== "")
              .map((i) => ({
                imageUrl: i.imageUrl,
                name: i.mosaic.id.name,
                namespaceId: i.mosaic.id.namespaceId,
              }));
            console.log(mosaics);
            this.oaMosaics = namespaces.map((n) => {
              const tmp = mosaics.find(
                (m) =>
                  m.name === n.mosaicId.name &&
                  m.namespaceId === n.mosaicId.namespaceId
              );
              return { quantity: n.quantity, ...tmp };
            }).filter(i => i.imageUrl);
          } catch (e) {
            throw new Error(e);
          } finally {
            this.loading = false;
          }
        }
      },
    }
  },

  methods: {
    login() {
      this.$auth.loginWithRedirect();
    },
    logout() {
      this.$auth.logout({
        returnTo:
          window.location.origin + "/" + window.location.pathname.split("/")[1],
      });
    },
  },
  components: {
    Loading,
  },
};
</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.toolbar__top.v-toolbar__content {
  color: #fff;
  text-decoration: none;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
