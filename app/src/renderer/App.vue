<template>
  <div id="#app" class="flex-container">

    <div class="header">
      <h1>PhotoBooth</h1>
    </div>

    <div class="camera" v-show="enabled">
      <div class="viewfinder" v-show="!img.preview"></div>

      <div class="preview" v-show="img.preview">
        <img v-bind:src="img.data" alt="">
      </div>
    </div>

    <div class="messages">
      <div v-show="msg.error">
        <div class="alert alert-danger">{{ msg.error }}</div>
      </div>

      <div v-show="msg.success">
        <div class="alert alert-success">{{ msg.success }}</div>
      </div>
    </div>


    <div class="flex-controls">
      <span id="power" class="control" v-bind:class="powerStatus()" v-on:click="toggleCamera()">
        <span class="glyphicon glyphicon-off"></span>
      </span>

      <span id="capture" class="control" v-on:click="takePhoto()">
        <span class="glyphicon glyphicon-camera"></span>
      </span>

      <span id="save" class="control" v-on:click="saveImg()">
        <span class="glyphicon glyphicon-floppy-disk"></span>
      </span>

      <span id="trash" class="control" v-on:click="reset()">
        <span class="glyphicon glyphicon-trash"></span>
      </span>
    </div>

  </div>
</template>

<script>
  var WebCam = require('webcamjs')
  var {dialog} = require('electron').remote
  var fs = require('fs')

  export default {
    data () {
      return {
        enabled: false,
        msg: {
          error: '',
          success: ''
        },
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
          booted: '',
          powerUp: '',
          powerDown: '',
          snapped: 'click.aif',
          saved: '',
          trashed: ''
        }
      }
    },

    mounted () {
      // this.initialiseCamera()
    },

    methods: {

      initialiseCamera () {
        WebCam.set(this.cameraOptions)
      },

      toggleCamera () {
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

        WebCam.snap(function (dataUri) {
          self.img = {
            data: dataUri,
            preview: true
          }
          self.play(self.audio.snapped)
        })
      },

      saveImg () {
        if (!this.enabled) {
          return
        }

        var self = this

        dialog.showSaveDialog(this.filters, function (fileName) {
          if (fileName === undefined) {
            this.msg.error = 'File not saved. No file name given.'
            return
          }

          fs.writeFile(fileName + '.jpg', self.processBase64Image(), function (err) {
            if (err) {
              self.msg.error = 'Cannot save the file'
            } else {
              self.msg.success = 'Image saved succesfully'
            }
          })
        })
      },

      processBase64Image () {
        var response = {}

        var matches = this.img.data.match(/^data:([A-Za-z-+/]+);base64,(.+)$/)

        if (matches.length !== 3) {
          return new Error('Invalid input string')
        }

        response.type = matches[1]
        response.data = new Buffer(matches[2], 'base64')

        return response.data
      },

      reset () {
        this.img = {
          preview: false,
          data: ''
        }

        this.msg = {
          error: '',
          success: ''
        }
      },

      play (file) {
        new Audio('./assets/audio/' + file).play()
      }
    }
  }
</script>

<style src="./assets/sass/app.scss" lang="scss"></style>
