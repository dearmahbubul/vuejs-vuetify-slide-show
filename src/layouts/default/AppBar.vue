<template>
  <v-layout>
    <v-app-bar app color="green" dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>LOGO</v-toolbar-title>
    </v-app-bar>
    <v-navigation-drawer v-model="drawer" absolute temporary>
      <v-list nav dense>
        <v-list-subheader>Menus</v-list-subheader>
        <v-divider></v-divider>
        <v-list density="compact">
          <v-list-item
            v-for="(item, i) in items"
            :key="i"
            :value="item"
            active-color="primary"
          >
            <router-link :to=item.link>
              <template v-slot:prepend>
                <v-icon :icon="item.icon"></v-icon>
              </template>
              <v-list-item-title>{{item.title}}</v-list-item-title>
            </router-link>
          </v-list-item>
        </v-list>
      </v-list>
    </v-navigation-drawer>
    <v-main style="min-height: 300px;">
      <router-view />
    </v-main>
  </v-layout>
</template>

<script>
export default {
  name: 'App',
  data: () => ({
    drawer: false,
    group: true,
    items: [
      {title: 'Dashboard', icon: 'mdi-view-dashboard', link: '/'},
      {title: 'Projects', icon: 'mdi-account-box', link: '/projects'},
      {title: 'Settings', icon: 'mdi-cog', link: '/'},
    ],
  }),
};
</script>
