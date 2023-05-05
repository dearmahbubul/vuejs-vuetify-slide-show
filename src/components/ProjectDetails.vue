<template>
  <h2>Hello world</h2>
</template>

<script>

import axios from "axios";
const baseUrl = "lang/api/v1";
export default {
  data: () => ({
    searchItem: "",
    dialog: false,
    loading: false,
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
    desserts: [],
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

      this.loading = true;
      await axios
        .get("http://jsonplaceholder.typicode.com/posts")
        // .get(`${baseUrl}/projects`)
        .then(response => {
          this.desserts = response.data
        })
        .catch(error => {
          console.log(error)
          if(error.message) {
            alert("Failed! " + error.message);
          }
          this.errored = true
        })
        .finally(() => this.loading = false)
    },

    editItem(item) {
      this.editedIndex = this.desserts.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.desserts.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    async deleteItemConfirm() {
      this.loading = true;
      const actionUrl = this.editedIndex > -1 ? `${baseUrl}/projects/${this.desserts[this.editedIndex].id}` : `${baseUrl}/projects`;

      await axios
        .delete(actionUrl, this.editedItem)
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
      // this.desserts.splice(this.editedIndex, 1)
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
          .put(`${baseUrl}/projects/${this.desserts[this.editedIndex].id}`, this.editedItem)
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

        // this.desserts.push(this.editedItem)

      this.close()
    },
  },
}
</script>
