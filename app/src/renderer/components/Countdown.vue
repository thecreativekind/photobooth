<template lang="html">
  <transition name="countdown">
    <div class="countdown-mask">
      <div class="countdown-wrapper">
        <div class="countdown-container">
          <div class="countdown-body">
              {{ counter }}
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  export default {

    props: ['startFrom'],

    data () {
      return {
        counter: ''
      }
    },

    mounted () {
      this.counter = this.startFrom
      this.countdown()
    },

    methods: {

      countdown () {
        var self = this

        var x = setInterval(function () {
          self.counter--
          if (self.counter < 0) {
            clearInterval(x)
            self.$emit('close')
          }
        }, 1000)
      }
    }
  }
</script>
