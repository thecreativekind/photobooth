<template>
  <div id="#app" class="flex-container">

    <div class="header">
      <h1>PhotoBooth</h1>

      <div class="modals">
        <settings v-if="showSettingsModal" @close="showSettingsModal = false"></settings>

        <save v-if="showSaveModal" @close="close()"></save>

        <countdown v-if="showCountdown" @close="snap()" :startFrom="countdown"></countdown>
      </div>
    </div>

    <div class="camera" v-show="enabled">
      <div class="viewfinder" v-show="!img.preview"></div>

      <div class="preview" v-show="img.preview">
        <img v-bind:src="img.data">
      </div>

    </div>

    <div class="flex-controls">
      <span id="power" class="control" v-bind:class="powerStatus()" v-on:click="toggleCamera()">
        <span class="glyphicon glyphicon-off"></span>
      </span>

      <span id="capture" class="control" v-on:click="initialiseCountdown()">
        <span class="glyphicon glyphicon-camera"></span>
      </span>

      <span id="save" class="control" v-on:click="saveImg()">
        <span class="glyphicon glyphicon-floppy-disk"></span>
      </span>

      <span id="trash" class="control" v-on:click="reset(true)">
        <span class="glyphicon glyphicon-trash"></span>
      </span>

      <span id="settings" class="control" @click="showSettingsModal = true">
        <span class="glyphicon glyphicon-cog"></span>
      </span>

    </div>

  </div>
</template>

<script>
  import Countdown from './components/Countdown'
  import Save from './components/Save'
  import Settings from './components/Settings'

  const axios = require('axios')
  const electron = require('electron')
  const fs = require('fs')
  const WebCam = require('webcamjs')
  const {dialog} = electron.remote

  var screenElectron = electron.screen
  var store = localStorage

  export default {

    components: {
      Countdown, Save, Settings
    },

    data () {
      return {
        enabled: false,
        showSettingsModal: false,
        showSaveModal: false,
        showCountdown: false,
        countdown: 3,
        filters: [
          { name: 'Images', extensions: ['jpeg'] }
        ],
        img: {
          data: '',
          preview: false
        },
        cameraOptions: {
          width: 720,
          height: 540,
          image_format: 'jpeg',
          jpeg_quality: 90
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
        var mainScreen = screenElectron.getPrimaryDisplay()

        WebCam.set({
          width: mainScreen.size.width / 2,
          height: mainScreen.size.height / 2,
          image_format: 'jpeg',
          jpeg_quality: 90
        })
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

      initialiseCountdown () {
        if (!this.enabled) {
          return
        }

        var self = this
        self.showCountdown = true
      },

      snap () {
        var self = this

        self.showCountdown = false

        WebCam.snap(function (dataUri) {
          self.img = {
            data: dataUri,
            preview: true
          }
          self.play(self.audio.snapped)
        })
      },

      saveImg () {
        var self = this

        if (this.img.data === '') {
          return
        }

        dialog.showSaveDialog(this.filters, function (fileName) {
          if (fileName === undefined) {
            return
          }

          self.writeFile(fileName)
          self.upload()
          self.play(self.audio.saved)
          self.showSaveModal = true
        })
      },

      writeFile (fileName) {
        var self = this

        fs.writeFile(fileName + '.jpg', self.processBase64Image(), function (error) {
          if (error) {
            console.log(error)
          }
        })
      },

      processBase64Image () {
        var matches = this.img.data.match(/^data:([A-Za-z-+/]+);base64,(.+)$/)

        if (matches.length !== 3) {
          return new Error('Invalid input string')
        }

        return new Buffer(matches[2], 'base64')
      },

      upload () {
        var self = this
        var url = store.getItem('endpoint')

        if (!url) {
          return
        }

        axios.post(url, {img: self.img.data})
        .then((response) => {
          self.showSaveModal = true
          console.log(response)
        })
        .catch((error) => {
          console.log(error)
        })

        console.log('uploading')
      },

      close () {
        this.showSaveModal = false
        this.reset(true)
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
      }
    }
  }
</script>

<style src="./assets/sass/app.scss" lang="scss"></style>
