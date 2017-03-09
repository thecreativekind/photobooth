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

  export default {

    components: {Upload},

    data () {
      return {
        enabled: false,
        showUploadModal: false,
        uploadUrl: config.endpoint,
        filters: [
          { name: 'Images', extensions: ['jpeg'] }
        ],
        img: {
          data: '',
          preview: false
        },
        cameraOptions: {
          width: mainScreen.size.width / 2,
          height: mainScreen.size.height / 2,
          image_format: 'jpeg',
          jpeg_quality: 90
        },
        audio: {
          booted: 'power-up.mp3',
          power: 'power-up.mp3',
          snapped: 'click.mp3',
          upoaded: 'complete.mp3',
          saved: 'saved.mp3',
          trashed: 'delete.mp3'
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
        var self = this

        if (!this.enabled) {
          return
        }

        WebCam.snap(function (dataUri) {
          self.img = {
            data: dataUri,
            preview: true
          }

          self.play(self.audio.snapped)
        })
      },

      saveImg () {
        if (this.img.data === '') {
          return
        }

        this.showUploadModal = true
      },

      processBase64Image () {
        var matches = this.img.data.match(/^data:([A-Za-z-+/]+);base64,(.+)$/)

        if (matches.length !== 3) {
          return new Error('Invalid input string')
        }

        return new Buffer(matches[2], 'base64')
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
        new Audio('./assets/audio/' + file).play()
      },

      close () {
        this.showUploadModal = false
        this.reset(true)
      }
    }
  }
</script>

<style src="./assets/sass/app.scss" lang="scss"></style>
