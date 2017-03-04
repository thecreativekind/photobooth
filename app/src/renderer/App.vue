<template>
  <div id="#app" class="flex-container">

    <div class="header flex-item">
      <h1>Photo Booth</h1>
    </div>

    <div class="camera flex-item">
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

    <div class="controls flex-item">
      <ul class="list-inline">
        <li>
          <span class="btn" v-bind:class="powerStatus()" v-on:click="toggleCamera()">
            <span class="glyphicon glyphicon-off"></span>
          </span>
        </li>

        <li>
          <span class="btn btn-default" v-bind:class="canSnap()" v-on:click="takePhoto()">
            <span class="glyphicon glyphicon-camera"></span>
          </span>
        </li>

        <li>
          <span class="btn btn-default" v-bind:class="canSave()" v-on:click="saveImg()">
            <span class="glyphicon glyphicon-floppy-disk"></span>
          </span>
        </li>

        <li>
          <span class="btn btn-default" v-bind:class="canSave()" v-on:click="reset()">
            <span class="glyphicon glyphicon-trash"></span>
          </span>
        </li>
      </ul>
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
          width: 800,
          height: 600,
          image_format: 'jpeg',
          jpeg_quality: 90
        }
      }
    },

    mounted () {
      this.initialiseCamera()
    },

    methods: {

      initialiseCamera () {
        WebCam.set(this.cameraOptions)
      },

      toggleCamera () {
        if (!this.enabled) {
          return this.enableCamera()
        }

        return this.disableCamera()
      },

      enableCamera () {
        this.enabled = true
        WebCam.attach('.viewfinder')
      },

      disableCamera () {
        this.enabled = false
        WebCam.reset()
      },

      powerStatus () {
        if (this.enabled) {
          return 'btn-danger'
        }

        return 'btn-default'
      },

      canSnap () {
        if (!this.enabled) {
          return 'disabled'
        }
      },

      canSave () {
        if (!this.img.preview) {
          return 'disabled'
        }
      },

      takePhoto () {
        var self = this

        WebCam.snap(function (dataUri) {
          self.img = {
            data: dataUri,
            preview: true
          }
        })
      },

      saveImg () {
        var self = this

        dialog.showSaveDialog(this.filters, function (fileName) {
          if (fileName === undefined) {
            this.msg.error = 'File not saved. No file name given.'
            return
          }

          fs.writeFile(fileName, self.processBase64Image(), function (err) {
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

        // this.msg = {
        //   error: '',
        //   success: ''
        // }
      }
    }
  }
</script>

<style src="./assets/sass/app.scss" lang="scss"></style>
