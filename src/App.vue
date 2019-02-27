<template>

  <div id="app">

    <div class="setting-bars">

      <div class="bar-element brightness">

        <h2 class="section-title">
          Brightness
        </h2>
        
        <div :class="{inactive: disableSlider}" class="slider-wrapper">
          <BrightnessContrastSlider v-model="imageBrightness" v-on:change="drawImage"/>
          <div class="apply-disable">&nbsp;</div>
        </div>

        <p>Slide to adjust image brightness!☀️</p>

      </div>

      <div class="bar-element contrast">

        <h2 class="section-title">
          Contrast
        </h2>

        <div :class="{inactive: disableSlider}" class="slider-wrapper">
          <BrightnessContrastSlider v-model="imageContrast" v-on:change="drawImage"/>
          <div class="apply-disable">&nbsp;</div>
        </div>

        <p>Slide to adjust image contrast! 🌓</p>

      </div>

    </div>

    <div class="user-section" ref="imageBox">

      <canvas id="viewport"></canvas>

      <div class="file-row clearfix">

        <span class="name-block">

          <label for="userImgFile">Name</label>
          <input type="text" id="userImgFile" value="FILE NAME ...">

        </span>

        <div class="upload-button-wrapper">
          <button id="upload-button">
            <svg height="20" width="30">
              <polygon points="11,4 20,18 2,18" stroke-linejoin="round" style="fill:#4A90E2;stroke:#4A90E2;stroke-width:4"></polygon>
            </svg>
            Upload
          </button>
          <input type="file" id="fileUpload" ref="myFiles" v-on:change="processPreview">
        </div>

      </div>

      <button :disabled="!!disableSlider" id="reset-button" v-on:click="imgReset">Reset</button>

    </div>
    
  </div>
</template>

<script>

import BrightnessContrastSlider from './components/BrightnessContrastSlider.vue';

export default {
  data: function(){
    return {
      imageSrc: require('@/assets/placeholder.png'),
      imageBrightness: 50,
      imageContrast: 50,
      imageWidth: 0,
      imageHeight: 0,
      targetImage: {
        type: Array
      },
      disableSlider: true
    }
  },
  watch: {
    imageBrightness: function(){
      this.imgLoad();
    },
    imageContrast: function(){
      this.imgLoad();
    }
  },
  name: 'app',
  components: {
    BrightnessContrastSlider
  },
  mounted:function(){
    this.imgLoad(true);
  },
  methods: {
    imgLoad:function(isLoad){
      this.imageWidth = this.$refs.imageBox.clientWidth;

      var base_image = new Image();
      base_image.src = this.imageSrc;
      this.targetImage = base_image;

      if (isLoad) {
        base_image.onload = (e) => {
          this.imageHeight = (base_image.height/(base_image.width/this.imageWidth));
          this.drawImage();
        }
      } else {
        this.drawImage();
      } 
    },

    drawImage: function(){
      var canvas = document.getElementById('viewport');
      var context = canvas.getContext('2d');
      canvas.width = this.imageWidth;
      canvas.height = this.imageHeight;

      var canvasBrightness = this.imageBrightness * 2;
      var canvasContrast = this.imageContrast / 50;

      context.drawImage(this.targetImage, 0, 0,this.imageWidth, this.imageHeight );

      var imageData = context.getImageData(0, 0, this.imageWidth, this.imageHeight);
      this.applyBrightnessContrast(imageData.data);

      context.putImageData(imageData, 0, 0);
    },

    applyBrightnessContrast:function(data) {
      var brightnessVal = this.imageBrightness * 2 - 100;

      for (var i = 0; i < data.length; i+= 4) {
          data[i] += 255 * (brightnessVal / 100);
          data[i+1] += 255 * (brightnessVal / 100);
          data[i+2] += 255 * (brightnessVal / 100);
      }

      var contrastVal = this.imageContrast * 2 - 100;
      var factor = (259.0 * (contrastVal + 255.0)) / (255.0 * (259.0 - contrastVal));

      for (var i = 0; i < data.length; i+= 4) {
        data[i] = this.limitColorValue(factor * (data[i] - 128.0) + 128.0);
        data[i+1] = this.limitColorValue(factor * (data[i+1] - 128.0) + 128.0);
        data[i+2] = this.limitColorValue(factor * (data[i+2] - 128.0) + 128.0);
      }
    },

    limitColorValue:function(value){
      if (value < 0) {
        value = 0;
      } else if (value > 255) {
        value = 255;
      } 
      return value;
    },

    imgReset:function(){
      this.imageBrightness = 50,
      this.imageContrast = 50
    },

    processPreview:function(event) {
      
      var input = event.target;
      this.disableSlider = true;

      if (input.files && input.files[0]) {
          
          var reader = new FileReader();
          
          reader.onload = (e) => {
              this.imageSrc = e.target.result;
              this.imgLoad(true);
              this.disableSlider = false;
              this.imgReset();
          }

          reader.readAsDataURL(input.files[0]);
          document.getElementById('userImgFile').value = input.files[0].name;
      }
    }
  }
}

</script>

<style src="@/assets/css/app.css"></style>
