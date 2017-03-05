<template lang="html">
  <transition name="modal">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">

          <div class="modal-header">
            <h3>B.E.A.Yootiful!</h3>
          </div>

          <div class="modal-body">
              Good job!
          </div>

          <div class="modal-footer">

            <slot name="footer">
              <span class="btn btn-default modal-default-button" @click="$emit('close')">
                CLOSE
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
        store.setItem('directory', document.getElementsByName('directory')[0].value)
        store.setItem('endpoint', document.getElementsByName('endpoint')[0].value)
        this.saved = true
      }
    }
  }
</script>
