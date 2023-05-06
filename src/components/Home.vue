<template>
  <v-item-group selected-class="bg-primary">
    <v-container>
      <v-row v-if="renderLoading">
        <v-col
          cols="12"
          md="6"
          v-for="n in 2" :key="n">
          <v-skeleton-loader
            class="mx-auto"
            elevation="12"
            type="image"
          ></v-skeleton-loader>
        </v-col>
      </v-row>
      <v-row v-else>
        <v-col
          :key="1"
          cols="12"
          md="6"
        >
          <router-link to="/projects" class="text-decoration-none">
            <v-item>
              <v-card
                class='d-flex align-center'
                color="lime"
                height="200"
              >
                <div
                  class="text-h3 flex-grow-1 text-center"
                >
                  <h4>Projects</h4>
                </div>
              </v-card>
            </v-item>
          </router-link>
        </v-col>
        <v-col
          :key="1"
          cols="12"
          md="6"
        >
          <router-link to="/projects" class="text-decoration-none">
            <v-item>
              <v-card
                class='d-flex align-center'
                color="primary"
                height="200"
              >
                <div
                  class="text-h3 flex-grow-1 text-center"
                >
                  <h4>Facebook</h4>
                </div>
              </v-card>
            </v-item>
          </router-link>
        </v-col>
      </v-row>
    </v-container>
  </v-item-group>
</template>

<script>
export default {
  data: () => ({
    renderLoading: false,
  }),
  mounted() {
    this.renderLoading = true
    setTimeout(() => this.renderLoading = false, 1000);
  }
}
</script>
