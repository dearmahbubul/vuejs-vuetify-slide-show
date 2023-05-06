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
      <v-row v-else-if="slides.length > 0">
        <v-col
          v-for="(item, i) in slides"
          :key="i"
          cols="12"
          md="6"
        >
          <v-item>
            <v-img
              :src="item.image ? item.image : `https://cdn.vuetifyjs.com/images/backgrounds/bg.jpg`"
              cover
              height="150"
              class="text-right pa-2"
            >
              <v-btn class="mr-2" icon="mdi-pencil" @click="editItem(item)"></v-btn>
              <v-btn color="error" icon="mdi-delete" @click="deleteItem(item)"></v-btn>

              <div
                class="flex-grow-1 text-center"
              >
                <v-chip
                  class="pa-10"
                  label
                >
                  <div class="">
                    <h3>{{i + 1}}</h3>
                    <h6>{{ item.text }}</h6>
                  </div>
                </v-chip>
              </div>
            </v-img>
          </v-item>
        </v-col>
      </v-row>
      <v-row v-else>
        <v-col
          cols="12"
        >
          <v-alert
            variant="outlined"
            type="error"
            prominent
            border="top"
          >
            There are no slides
          </v-alert>
        </v-col>
      </v-row>
    </v-container>
    <v-btn
      v-if="slides.length > 0"
      style="right: 0; top: 0"
      class="ma-2 position-absolute"
      color="success"
      icon="mdi-play"
    ></v-btn>
  </v-item-group>
  <v-dialog
    v-model="dialog"
    scrollable
    max-width="500px"
  >

    <template v-slot:activator="{ props }">
      <v-btn
        v-bind="props"
        style="right: 0; bottom: 0"
        class="ma-2 position-absolute"
        color="indigo"
        icon="mdi-plus"
      ></v-btn>
    </template>
    <v-form @submit.prevent="save">
      <v-card>
        <v-card-title>
          <span class="text-h5">{{ formTitle }}</span>
        </v-card-title>

        <v-card-text style="max-height: 600px">
          <v-container>
            <v-row>
              <v-col
                cols="12"
              >
                <v-textarea
                  counter
                  clearable
                  auto-grow
                  autocomplete="text"
                  clear-icon="mdi-close-circle"
                  required
                  label="Slide Text"
                  :rules="rules"
                  v-model="editedItem.text"
                  variant="outlined"
                ></v-textarea>
              </v-col>
              <v-col
                cols="12"
              >
                <v-file-input
                  show-size
                  prepend-icon=""
                  :rules="fileRules"
                  accept="image/*"
                  @change.prevent="selectImage"
                  placeholder="Pick an image"
                  prepend-inner-icon="mdi-camera"
                  label="Image"
                  variant="outlined"
                ></v-file-input>
              </v-col>
              <v-col
                cols="12"
              >
                <v-text-field
                  type="number"
                  name="imageWidth"
                  required
                  v-model="editedItem.imageWidth"
                  label="Image Width"
                  variant="outlined"
                ></v-text-field>
              </v-col>
              <v-col
                cols="12"
              >
                <v-text-field
                  type="number"
                  name="imageHeight"
                  required
                  v-model="editedItem.imageHeight"
                  label="Image Height"
                  variant="outlined"
                ></v-text-field>
              </v-col>
              <v-col
                cols="12"
                v-if="editedItem.image"
              >
                <v-img
                  :width="editedItem.imageWidth"
                  aspect-ratio="16/9"
                  cover
                  :src="editedItem.image"
                ></v-img>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="blue-darken-1"
            variant="text"
            @click="close"
          >
            Cancel
          </v-btn>
          <v-btn
            color="blue-darken-1"
            variant="text"
            type="submit"
            :disabled="loading"
          >
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-form>
  </v-dialog>
  <v-dialog v-model="dialogDelete" max-width="500px">
    <v-card>
      <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancel</v-btn>
        <v-btn color="blue-darken-1" variant="text" @click="deleteItemConfirm" :disabled="loading">OK</v-btn>
        <v-spacer></v-spacer>
      </v-card-actions>
    </v-card>
  </v-dialog>

</template>

<script>

import axios from "axios";
const baseUrl = "/lang/api/v1";
export default {
  data: () => ({
    fileRules: [
      value => {
        return !value || !value.length || value[0].size < 2000000 || 'Image size should be less than 2 MB!'
      },
    ],
    rules: [v => v.length <= 500 || 'Max 500 characters'],
    searchItem: "",
    dialog: false,
    loading: false,
    renderLoading: false,
    dialogDelete: false,
    slides: [],
    editedIndex: -1,
    editedItem: {
      text: '',
      image: '',
      imageWidth: '',
      imageHeight: ''
    },
    defaultItem: {
      text: '',
      image: '',
      imageWidth: '',
      imageHeight: ''
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Create New Slide' : 'Edit Slide'
    },
    projectId() {
      return this.$route.params.id
    }
  },

  watch: {
    dialog(val) {
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },
  },

  created() {
    this.initialize()
  },

  methods: {
    async initialize() {
      this.renderLoading = true;
      await axios
        .get(`${baseUrl}/projects/${this.projectId}/slides`)
        .then(response => {
          this.slides = response.data
        })
        .catch(error => {
          console.log(error)
          if(error.message) {
            alert("Failed! " + error.message);
          }
        })
        .finally(() => this.renderLoading = false)
    },

    editItem(item) {
      this.editedIndex = this.slides.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.slides.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    async deleteItemConfirm() {
      this.loading = true;

      await axios
        .delete(`${baseUrl}/projects/${this.projectId}/slides/${this.slides[this.editedIndex].id}`)
        .then(response => {
          console.log(response);
          this.initialize();
        })
        .catch(error => {
          console.log(error)
          if(error.message) {
            alert("Failed! " + error.message);
          }
        })
        .finally(() => this.loading = false)
      this.closeDelete()
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    selectImage(e) {
      let file = e.target.files[0];
      //console.log(file);
      let reader = new FileReader();

      reader.onloadend = (file) => {
        // console.log('RESULT', reader.result)
        this.editedItem.image = reader.result;
      }
      this.editedItem.imageHeight = 125;
      this.editedItem.imageWidth = 125;
      reader.readAsDataURL(file);
    },

    async save() {
      this.loading = true;

      if(this.editedIndex > -1) {
        await axios
          .put(`${baseUrl}/projects/${this.projectId}/slides/${this.slides[this.editedIndex].id}`, this.editedItem)
          .then(() => {
            this.initialize();
          })
          .catch(error => {
            if(error.message) {
              alert("Failed! " + error.message);
            }
          })
          .finally(() => this.loading = false)
      } else {

        await axios
          .post(`${baseUrl}/projects/${this.projectId}/slides`, this.editedItem)
          .then(response => {
            const responseData = response.data;
            console.log(responseData);
            this.initialize();
          })
          .catch(error => {
            if(error.message) {
              alert("Failed! " + error.message);
            }
          })
          .finally(() => this.loading = false)
      }
      this.close()
    },
  },
}
</script>
