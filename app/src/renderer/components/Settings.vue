<template lang="html">
  <transition name="modal">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">

          <div class="modal-header">
            <h3>Settings</h3>
          </div>

          <div class="modal-body">

            <slot name="body">
              <div class="form">
                <div class="form-group spacer-bottom-5x">
                  <div class="">
                    <span class="pull-right" v-show="saved">
                      <span class="glyphicon glyphicon-ok text-success"></span>
                    </span>
                    <label for="directory">Default Directory</label>
                  </div>
                  <input type="text" name="directory" v-bind:value="directory" class="form-control">
                </div>

                <div class="form-group">
                  <div class="">
                    <span class="pull-right" v-show="saved">
                      <span class="glyphicon glyphicon-ok text-success"></span>
                    </span>
                    <label for="endpoint">API Endpoint</label>
                  </div>
                  <input type="text" name="endpoint" v-bind:value="endpoint" class="form-control">
                </div>

              </div>
            </slot>
          </div>

          <div class="modal-footer">

            <slot name="footer">
              <span class="btn btn-default modal-default-button" @click="$emit('close')">
                CLOSE
              </span>

              <span class="btn btn-primary modal-default-button" @click="persistData()">
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
  var store = localStorage

  export default {

    data () {
      return {
        directory: store.getItem('directory'),
        endpoint: store.getItem('endpoint'),
        saved: false
      }
    },

    created () {
      this.saved = false
    },

    methods: {
      persistData () {
        var directory = document.getElementsByName('directory')[0].value
        var endpoint = document.getElementsByName('endpoint')[0].value

        directory ? store.setItem('directory', directory) : store.removeItem('directory')
        endpoint ? store.setItem('endpoint', endpoint) : store.removeItem('endpoint')

        this.saved = true
      }
    }
  }
</script>
