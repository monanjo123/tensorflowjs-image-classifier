<template>
  <div class="camera">
      <video ref='video' autoplay playsinline muted width="224" height="224" class="feed"/>
  </div>
</template>

<script>
import Vue from 'vue'
import { MdButton, MdCard } from 'vue-material/dist/components'
import 'vue-material/dist/vue-material.min.css'
import 'vue-material/dist/theme/default.css'

Vue.use(MdButton)
Vue.use(MdCard)


export default {
  name: 'Webcam',
  async beforeMount() {
      await this.setupWebcam()
  },
  methods: {
      setupWebcam: async() => {
            return new Promise((resolve, reject) => {
              const navigatorAny = navigator;
              navigator.getUserMedia = navigator.getUserMedia || navigatorAny.webkitGetUserMedia || navigatorAny.mozGetUserMedia || navigatorAny.msGetUserMedia
              if (navigator.getUserMedia) {
                  navigator.getUserMedia(
                      {video: true},
                      stream => {
                          const webcamElement = document.querySelector('video')
                          console.log('setup')
                          webcamElement.srcObject = stream
                          webcamElement.addEventListener('loadeddata', ()=>resolve(), false)
                      },
                      error => reject())
              } else {
                  reject()
              }
          })
      }
  }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* .camera {
    box-sizing: border-box;
} */
.camera .feed {
    height: auto;
    width: auto;
    display: block;
    margin: auto;
    background-color: #171717;
    box-shadow: 6px 6px 12px 0px rgba(0, 0, 0, 0.35)
}
</style>
