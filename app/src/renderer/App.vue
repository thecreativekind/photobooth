<template>
  <div id="#app" class="flex-container">

    <div class="header">
      <h1>PhotoBooth</h1>

      <div class="modals">
        <upload :url="uploadUrl" :img="img" v-if="showUploadModal" @close="close()"></upload>
      </div>
    </div>

    <div class="camera" v-show="enabled">
      <div class="viewfinder" v-show="!img.preview"></div>

      <div class="preview" v-show="img.preview">
        <img v-bind:src="img.data" alt="">
      </div>
    </div>

    <div class="flex-controls">
      <span id="power" class="control" v-bind:class="powerStatus()" v-on:click="toggleCamera()">
        <span class="glyphicon glyphicon-off"></span>
      </span>

      <span id="capture" class="control" v-on:click="takePhoto()">
        <span class="glyphicon glyphicon-camera">
          <audio id="capture-audio" src="./assets/audio/click.mp3"></audio>
        </span>
      </span>

      <span class="control">
        {{ countdown }}
      </span>

      <span id="save" class="control" v-on:click="saveImg()">
        <span class="glyphicon glyphicon-floppy-disk"></span>
      </span>

      <span id="trash" class="control" v-on:click="reset(true)">
        <span class="glyphicon glyphicon-trash"></span>
      </span>

    </div>

  </div>
</template>

<script>
  import Upload from './components/Upload'
  const WebCam = require('webcamjs')
  const Electron = require('electron')
  const config = require('../config')

  var mainScreen = Electron.screen.getPrimaryDisplay()
  console.log(mainScreen.size.width, mainScreen.size.height)

  export default {

    components: {Upload},

    data () {
      return {
        enabled: false,

        showUploadModal: false,

        uploadUrl: config.endpoint,

        countdown: '',

        filters: [{
          name: 'Images',
          extensions: ['jpeg']
        }],

        img: {
          data: '',
          preview: false
        },

        cameraOptions: {
          width: mainScreen.size.width / 2,
          height: mainScreen.size.height / 2,
          image_format: 'jpeg',
          jpeg_quality: 100
        },

        audio: {
          booted: './assets/audio/power-up.mp3',
          power: './assets/audio/power-up.mp3',
          snapped: './assets/audio/click.mp3',
          upoaded: './assets/audio/complete.mp3',
          saved: './assets/audio/saved.mp3',
          trashed: './assets/audio/delete.mp3'
        }
      }
    },

    methods: {

      initialiseCamera () {
        WebCam.set(this.cameraOptions)
      },

      toggleCamera () {
        this.play(this.audio.power)

        return this.enabled ? this.disableCamera() : this.enableCamera()
      },

      enableCamera () {
        this.initialiseCamera()
        this.enabled = true
        WebCam.attach('.viewfinder')
      },

      disableCamera () {
        this.enabled = false
        this.img = {
          preview: false,
          data: ''
        }
        WebCam.reset()
      },

      powerStatus () {
        if (this.enabled) {
          return 'active'
        }
      },

      takePhoto () {
        if (!this.enabled) {
          return
        }

        var self = this

        var countdown = 3

        var x = setInterval(function () {
          self.countdown = countdown
          countdown--
          if (countdown < 0) {
            WebCam.snap(function (dataUri) {
              self.img = {
                data: dataUri,
                preview: true
              }

              self.play(self.audio.snapped)
            })

            clearInterval(x)
            self.countdown = ''
          }
        }, 1000)
      },

      saveImg () {
        if (this.img.data === '') {
          return
        }

        this.showUploadModal = true
      },

      reset (isDeleting = null) {
        if (isDeleting) {
          this.play(this.audio.trashed)
        }

        this.img = {
          preview: false,
          data: ''
        }
      },

      play (file) {
        new Audio(file).play()
      },

      close () {
        this.showUploadModal = false
        this.reset(true)
      }
    }
  }
</script>

<style src="./assets/sass/app.scss" lang="scss"></style>
