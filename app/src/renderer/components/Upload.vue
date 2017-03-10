<template lang="html">
  <transition name="modal">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">

          <div class="modal-header">
            <h3>{{ title }}</h3>
          </div>

          <div class="modal-body">
            <p v-show="msg != ''">{{ msg }}</p>

            <div id="progress"></div>

            <div class="form" v-show="!uploading">
              <div class="form-group spacer-bottom-5x">
                <div class="">
                  <span class="pull-right" v-show="saved">
                    <span class="glyphicon glyphicon-ok text-success"></span>
                  </span>
                  <label for="directory">Email</label>
                </div>
                <input type="text" name="email" v-model="email" class="form-control">
                <div class="spacer-top-1x alert alert-danger" v-show="error != ''">
                  <span class="glyphicon glyphicon-warning-sign"></span>
                  <small>{{ error }}</small>
                </div>
              </div>
            </div>
          </div>

          <div class="modal-footer">

            <slot name="footer">
              <span class="btn btn-default modal-default-button" @click="$emit('close')">
                CLOSE
              </span>

              <span class="btn btn-primary modal-default-button" @click="upload()" v-show="!uploading">
                SAVE
              </span>
            </slot>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  const axios = require('axios')
  const validator = require('email-validator')
  const ProgressBar = require('progressbar.js')

  export default {

    props: ['url', 'img'],

    data () {
      return {
        email: '',
        saved: false,
        uploading: false,
        error: '',
        title: 'Who Are You?',
        msg: ''
      }
    },

    created () {
      this.saved = false
    },

    methods: {
      upload () {
        if (!validator.validate(this.email)) {
          this.error = 'Invalid email address'

          return
        }

        this.uploading = true

        var bar = new ProgressBar.Line('#progress', {
          easing: 'easeInOut',
          color: '#42b983'
        })

        var config = {
          onUploadProgress: function (progress) {
            bar.animate(progress.loaded / progress.total)
          }
        }

        axios.post(this.url, {
          img: this.img.data,
          email: this.email
        }, config)
          .then(response => {
            this.title = 'Success'
            this.msg = 'You have successfully uploaded your image!'
          })
          .catch(error => {
            console.log('Error', error)
          })
      }

    }
  }
</script>
