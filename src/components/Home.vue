<template>
  <v-item-group selected-class="bg-primary">
    <v-container>
      <v-row>
        <v-col
          :key="1"
          cols="12"
          md="6"
        >
          <router-link to="/projects">
            <v-item>
              <v-card
                class='d-flex align-center'
                dark
                height="200"
              >
                <div
                  class="text-h3 flex-grow-1 text-center"
                >
                  Projects
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
          <router-link to="/projects">
            <v-item>
              <v-card
                class='d-flex align-center'
                dark
                height="200"
              >
                <div
                  class="text-h3 flex-grow-1 text-center"
                >
                  Facebook
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

</script>
