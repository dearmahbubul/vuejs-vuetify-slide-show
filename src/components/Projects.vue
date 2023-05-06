<template>
  <v-container v-if="renderLoading">
    <v-skeleton-loader
      class="mx-auto"
      elevation="12"
      type="table-heading, table-thead, table-row, table-tfoot"
    ></v-skeleton-loader>
  </v-container>
  <v-container v-else>
    <v-data-table
      :headers="headers"
      :items="projects"
      :search="searchItem"
      :loading="loading"
      loading-text="Loading... Please wait"
      :sort-by="[{ key: 'name', order: 'asc' }]"
      class="elevation-1"
      item-value="name"
    >
      <template v-slot:top>
        <v-toolbar
          dense
          floating
        >
          <v-toolbar-title>
            Projects
          </v-toolbar-title>
          <v-divider
            class="mx-4"
            inset
            vertical
          ></v-divider>
          <v-spacer></v-spacer>
          <v-dialog
            v-model="dialog"
            max-width="500px"
          >

            <template v-slot:activator="{ props }">
              <v-text-field
                class="mr-2"
                v-model="searchItem"
                prepend-icon="mdi-magnify"
                label="Search Projects"
                single-line
                hide-details
              ></v-text-field>
              <v-btn
                border
                prepend-icon="mdi-plus"
                v-bind="props"
                color="primary"
                size="small"
                variant="elevated"
              >
                Create New Project
              </v-btn>
            </template>
            <v-form @submit.prevent="save">
              <v-card>
                <v-card-title>
                  <span class="text-h5">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col
                        cols="12"
                      >
                        <v-text-field
                          name="name"
                          required
                          v-model="editedItem.name"
                          label="Project Name"
                          placeholder="Enter project name"
                        ></v-text-field>
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
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <router-link :to="'/projects/' + item.raw.id">
          <v-icon
            size="small"
            class="me-2"
            color="primary"
          >
            mdi-eye
          </v-icon>
        </router-link>
        <v-icon
          size="small"
          class="me-2"
          color="primary"
          @click="editItem(item.raw)"
        >
          mdi-pencil
        </v-icon>
        <v-icon
          size="small"
          color="error"
          @click="deleteItem(item.raw)"
        >
          mdi-delete
        </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn
          color="primary"
          class="mt-2 ml-2"
          prepend-icon="mdi-refresh"
          @click="initialize"
        >
          Reset
        </v-btn>
      </template>
    </v-data-table>
  </v-container>

</template>

<script>

import axios from "axios";
const baseUrl = "/lang/api/v1";
export default {
  data: () => ({
    searchItem: "",
    dialog: false,
    loading: false,
    renderLoading: false,
    dialogDelete: false,
    headers: [
      {
        title: 'Slideshow Name',
        align: 'start',
        sortable: true,
        key: 'name',
      },
      {title: 'Actions', key: 'actions', sortable: false},
    ],
    projects: [],
    editedIndex: -1,
    editedItem: {
      name: '',
    },
    defaultItem: {
      name: '',
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Create New Project' : 'Edit Project'
    },
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
        .get(`${baseUrl}/projects`)
        .then(response => {
          this.projects = response.data
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
      this.editedIndex = this.projects.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.projects.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    async deleteItemConfirm() {
      this.loading = true;

      await axios
        .delete(`${baseUrl}/projects/${this.projects[this.editedIndex].id}`)
        .then(response => {
          const responseData = response.data;
          console.log(responseData);
          this.initialize();
        })
        .catch(error => {
          console.log(error)
          if(error.message) {
            alert("Failed! " + error.message);
          }
          this.errored = true
        })
        .finally(() => this.loading = false)
      // this.projects.splice(this.editedIndex, 1)
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

    async save() {
      console.log('save')
      this.loading = true;

      if(this.editedIndex > -1) {
        await axios
          .patch(`${baseUrl}/projects/${this.projects[this.editedIndex].id}`, this.editedItem)
          .then(response => {
            const responseData = response.data;
            console.log(responseData);
            this.initialize();
          })
          .catch(error => {
            console.log(error)
            if(error.message) {
              alert("Failed! " + error.message);
            }
            this.errored = true
          })
          .finally(() => this.loading = false)
      } else {
        await axios
          .post(`${baseUrl}/projects`, this.editedItem)
          .then(response => {
            const responseData = response.data;
            console.log(responseData);
            this.initialize();
          })
          .catch(error => {
            console.log(error)
            if(error.message) {
              alert("Failed! " + error.message);
            }
            this.errored = true
          })
          .finally(() => this.loading = false)
      }

        // this.projects.push(this.editedItem)

      this.close()
    },
  },
}
</script>
