<template>
  <div id="app">
      <loading :active.sync="isLoading"></loading>
      <md-card>
        <md-card-media class="webcam">
          <Webcam ref='webcam'/>
        </md-card-media>
        <md-card-actions>
          <md-card-media-actions>
            <md-card-area>
              <md-card-header>
                <h2 class="md-title">Real Time Image Classifier</h2>
                <div class="md-subhead">
                  Add tags to real time video.
                </div>
              </md-card-header>
              <hr>
              <md-card-content>
                  <h1 class="md-title">Prediction: {{ prediction }}</h1>
                  <h1 class="md-title">Confidence: {{ confidence }}%</h1>
              </md-card-content>
              <hr>
              <md-card-content>
              </md-card-content>
              <md-card-content>
                <md-field>
                    <label>What's on the Camera???</label>
                    <md-input v-model="label" class="md-"></md-input>
                    <md-button class="md-raised md-primary" @click.prevent="addExample">Train</md-button>
                </md-field>
              </md-card-content>
            </md-card-area>
          </md-card-media-actions>
        </md-card-actions>
    </md-card>
  </div>
</template>

<script>
import Webcam from './components/Camera.vue'
import Vue from 'vue'
import * as mobilenet from '@tensorflow-models/mobilenet';
import * as knnClassifier from '@tensorflow-models/knn-classifier';
import * as tf from '@tensorflow/tfjs'
import { MdButton, MdCard, MdField } from 'vue-material/dist/components'
import Loading from 'vue-loading-overlay';

import 'vue-loading-overlay/dist/vue-loading.css';
import 'vue-material/dist/vue-material.min.css'
import 'vue-material/dist/theme/default.css'

Vue.use(MdButton)
Vue.use(MdCard)
Vue.use(MdField)

const classifier = knnClassifier.create();
let net;
let webcamElement;
let data_loaded = false
let pred;

export default {
  name: 'app',
  data() {
    return {
      prediction: '',
      confidence: '',
      isLoading: true,
    }
  },
  components: {
    Webcam,
    Loading
  },
  methods: {
    async get_prelim_data() {
      let frame_count = 0
      const interval = setInterval(async() => {
        if (data_loaded == false) {
          this.addExample()
          await tf.nextFrame()
          frame_count += 1
        }          
        //console.log(frame_count)

        if (frame_count == 50) {
          data_loaded = true
          clearInterval(interval)
          this.isLoading = false;
        }
        }, 50)
      
    },
    addExample() {
      let classId;
      if (pred) {
        clearInterval(pred)
      }
      this.label ?  classId = this.label : classId = 'No Action'
      //console.log(classId)
      // Get the intermediate activation of MobileNet 'conv_preds' and pass that to the KNN classifier.
      const activation = net.infer(webcamElement, 'conv_preds')
      // Pass the intermediate activation to the classifier.
      classifier.addExample(activation, classId)
      this.run_prediction()
       
    },
    async run_prediction() {
      // Define a recursive function
      pred = setInterval(async() => {
        if (classifier.getNumClasses() > 0) {
              const activation = net.infer(webcamElement, 'conv_preds')
              await classifier.predictClass(activation).then(response => {
                //console.log(response.label)
                this.prediction = response.label
                const probability = response.confidences[response.label] * 100 
                this.confidence = probability.toFixed(2)
              },
              error => console.log(error))
        }
            await tf.nextFrame()
      }, 50)
    }
  },
  async created() {
    console.log('Loading mobilenet . . .')
      // Load Model
    net = await mobilenet.load()
    console.log('Successfully loaded model')
    await this.get_prelim_data()
    await this.run_prediction()
  },
  mounted() {
    webcamElement = this.$refs.webcam.$refs.video
  }
}
</script>

<style scoped>
  .md-card {
    width: 80vw;
    height: 80vh;
    padding: 30px;
    margin: auto;
    box-sizing: border-box;
  }
  
  .webcam {
    position: absolute;
    height: auto;
    width: auto;
    display: block;
    left: 30px;
    bottom: 20px;
  }


</style>
