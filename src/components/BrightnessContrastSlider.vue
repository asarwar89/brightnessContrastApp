<template>
  <div>
    <vue-slider :value="value" v-on:input="$emit('input', $event)"></vue-slider>
  </div>
</template>

<script>

  import vueSlider from 'vue-slider-component';

  export default {
    props: {
      value: {
        type: Number
      }
    },
    components: {
      vueSlider
    }
  }

</script>
