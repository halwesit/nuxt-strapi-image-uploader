<template>
  <v-layout column justify-center align-center>
    <v-row>
      <v-col>
        <v-card v-if="!isUpdate" max-width="400" min-width="400">
          <v-progress-linear
            color="blue darken-2"
            indeterminate
            rounded
            height="6"
            v-if="waitting"
          ></v-progress-linear>
          <v-card-title>Insert Form</v-card-title>
          <v-card-text>
            <template>
              <v-form ref="form" v-model="valid" lazy-validation>
                <v-text-field
                  v-model="name"
                  :rules="nameRules"
                  label="Image Name"
                  required
                ></v-text-field>

                <v-text-field
                  v-model="photographer"
                  :rules="photographerRules"
                  label="Photographer Name"
                  required
                ></v-text-field>

                <v-file-input
                  v-model="image"
                  :rules="imageRules"
                  label="Select an Image"
                  accept="image/*"
                  :progress="waitting"
                  @change="onFileSelected"
                ></v-file-input>

                <v-btn
                  :disabled="!valid"
                  color="success"
                  class="mr-4"
                  @click="upload"
                >
                  Insert
                </v-btn>
              </v-form>
            </template>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col v-if="isUpdate">
        <v-card max-width="400" min-width="400">
          <v-progress-linear
            color="blue darken-2"
            indeterminate
            rounded
            height="6"
            v-if="waitting"
          ></v-progress-linear>
          <v-card-title>Update Form</v-card-title>
          <v-card-text>
            <template>
              <v-form ref="form" v-model="valid" lazy-validation>
                <v-text-field
                  v-model="name"
                  :rules="nameRules"
                  label="Image Name"
                  required
                ></v-text-field>

                <v-text-field
                  v-model="photographer"
                  :rules="photographerRules"
                  label="Photographer Name"
                  required
                ></v-text-field>

                <v-btn
                  :disabled="!valid"
                  color="success"
                  class="mr-4"
                  @click="update"
                >
                  Update
                </v-btn>

                <v-btn color="success" class="mr-4" @click="returnBack">
                  Back to insert
                </v-btn>
              </v-form>
            </template>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-progress-linear
      color="blue darken-2"
      indeterminate
      rounded
      height="6"
      v-if="loading"
    ></v-progress-linear>

    <v-row v-if="imageList">
      <v-col v-for="(img, index) in imageList.data" :key="index">
        <v-card class="mx-auto" width="300">
          <v-img
            class="white--text align-end"
            height="200px"
            :src="'http://localhost:1337' + img.image.url"
          >
          </v-img>
          <v-card-title>{{ img.name }}</v-card-title>

          <v-card-subtitle class="pb-0">{{ img.photographer }}</v-card-subtitle>

          <v-card-text class="text--primary">
            <div>{{ myDateFormat(img.createdAt) }}</div>
          </v-card-text>

          <v-card-actions>
            <v-btn color="blue" text @click="updateForm(img)">
              update
            </v-btn>

            <v-btn color="red" text @click="deleteimg(img)">
              Delete
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-layout>
</template>

<script>
import moment from 'moment'

export default {
  data: () => ({
    valid: true,
    name: '',
    nameRules: [v => !!v || 'Image Name is required'],
    photographer: '',
    photographerRules: [v => !!v || 'Photographer name is required'],
    imageRules: [v => !!v || 'Image is required'],
    image: null,
    selectedImage: null,
    waitting: false,
    loading: true,
    imageList: [],
    isUpdate: false,
    currentImg: null
  }),
  mounted() {
    this.show()
  },
  methods: {
    onFileSelected(event) {
      this.selectedImage = event
    },
    myDateFormat(d) {
      return moment(String(d)).format('MM/DD/YYYY')
    },
    async upload() {
      if (this.$refs.form.validate()) {
        this.waitting = true
        var formdata = new FormData()
        formdata.append('files', this.selectedImage, this.selectedImage.name)
        var requestOptions = {
          method: 'POST',
          body: formdata,
          redirect: 'follow'
        }
        await fetch('http://localhost:1337/upload', requestOptions)
          .then(resp => resp.text())
          .then(res => {
            this.waitting = false
            this.addData(res)
          })
          .catch(error => console.log('error', error))
      }
    },
    async addData(image) {
      this.waitting = true
      this.loading = true
      let imgRes = JSON.parse(image)
      let imageID = imgRes[0].id
      var myHeaders = new Headers()
      myHeaders.append('Content-Type', 'application/json')
      var raw = JSON.stringify({
        name: this.name,
        photographer: this.photographer,
        image: imageID
      })

      var requestOptions = {
        method: 'POST',
        headers: myHeaders,
        body: raw,
        redirect: 'follow'
      }

      await fetch('http://localhost:1337/galleries', requestOptions)
        .then(response => response.text())
        .then(resu => {
          this.waitting = false
          console.log(resu)
          this.show()
          this.reset()
        })
        .catch(err => console.log('error', err))
    },

    updateForm(img) {
      this.reset()
      this.isUpdate = true
      this.currentImg = img
      this.name = this.currentImg.name
      this.photographer = this.currentImg.photographer
    },
    returnBack() {
      this.isUpdate = false
      this.currentImg = null
      this.reset()
    },
    async update() {
      this.waitting = true
      this.loading = true
      if (this.$refs.form.validate()) {
        var myHeaders = new Headers()
        myHeaders.append('Content-Type', 'application/json')

        var raw = JSON.stringify({
          name: this.name,
          photographer: this.photographer
        })

        var requestOptions = {
          method: 'PUT',
          headers: myHeaders,
          body: raw,
          redirect: 'follow'
        }

        await fetch(
          'http://localhost:1337/galleries/' + this.currentImg.id,
          requestOptions
        )
          .then(response => response.text())
          .then(result => {
            console.log(result)
            this.show()
            this.isUpdate = false
            this.loading = false
            this.waitting = false
            this.currentImg = null
            this.reset()
          })
          .catch(error => console.log('error', error))
      }
    },
    async deleteimg(img) {
      this.loading = true
      var requestOptions = {
        method: 'DELETE',
        redirect: 'follow'
      }

      await fetch('http://localhost:1337/galleries/' + img.id, requestOptions)
        .then(response => response.text())
        .then(result => {
          var requestOptions = {
            method: 'DELETE',
            redirect: 'follow'
          }

          fetch(
            'http://localhost:1337/upload/files/' + img.image.id,
            requestOptions
          )
            .then(response => response.text())
            .then(result => {
              this.show()
            })
            .catch(error => console.log('error', error))
        })
        .catch(error => console.log('error', error))
    },
    async show() {
      await this.$axios.get('http://localhost:1337/galleries').then(imgInfo => {
        this.imageList = imgInfo
        this.loading = false
      })
    },
    reset() {
      this.$refs.form.reset()
    }
  }
}
</script>
