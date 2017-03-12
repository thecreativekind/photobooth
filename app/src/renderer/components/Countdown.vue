<template lang="html">
  <transition name="countdown">
    <div class="countdown-mask">
      <div class="countdown-wrapper">
        <div class="countdown-container">
          <div class="countdown-body">
              {{ countdown }}
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  export default {

    props: ['countdown'],

    methods: {
      //
    }
  }
</script>
