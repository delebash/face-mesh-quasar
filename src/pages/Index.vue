<template>
  <q-page-container class="no-padding">
    <q-page>
      <q-tab-panels v-model="tab" swipeable animated style="width: 100%">

        <q-tab-panel class="text-center" name="preview">
          <q-btn ref='preview' v-on:click="Startpreview" color="white" text-color="black" label="Start Preview"/>
          <q-btn ref='record' v-on:click="Startrecord" color="white" text-color="black" label="Start Recording"/>
          <q-checkbox @input="Checkchange" v-model="checked" label="Triangulation"/>

          <div class="text-center">
            <br/>
            <canvas id="output"></canvas>
            <br/>
            <video id="video"></video>
          </div>
        </q-tab-panel>

        <q-tab-panel class="no-padding" swipeable name="settings">
          <div class="">
            <q-select v-model="select"
                      :options="options"
                      label="Select Camera"
            />
          </div>
        </q-tab-panel>
      </q-tab-panels>
    </q-page>
    <q-footer>
      <q-tabs v-model="tab">
        <q-tab name="preview" label="Preview"/>
        <q-tab name="settings" label="Settings"/>
      </q-tabs>
    </q-footer>
  </q-page-container>
</template>
<script>

let isMobile, frameClount = 0, path, logfile, isElectron, cameraId, model, ctx, videoWidth, videoHeight, video, canvas,
  currentStream,
  record = false,
  scatterGLHasInitialized = false, scatterGL, Triangulationmesh = false

const VIDEO_SIZE = 500;
import '@tensorflow/tfjs'
// import '@tensorflow/tfjs-node-gpu';


import fs from 'fs'
import JSONStream from 'jsonstream-next'

let jsonwriter = JSONStream.stringify();

import {Platform} from 'quasar'
import * as facemesh from '@tensorflow-models/facemesh'
import TRIANGULATION from '../js/triangulation'
import prependFile from 'prepend-file';

export default {
  data() {
    return {
      tab: 'preview',
      select: null,
      checked: false,
      options: []
    }
  },
  async mounted() {
    this.options = await getCameraList()
    isMobile = Platform.is.mobile
    isElectron = Platform.is.electron
  },
  methods: {

    Checkchange: function () {
      Triangulationmesh = this.checked
    },
    Startpreview: function () {

      frameClount = 0
      if (typeof this.select === 'undefined' || this.select === null) {
        cameraId = null
        alert('Please Select a camera')
      } else {
        cameraId = this.select.value
        Triangulationmesh = this.val
        let el = this.$refs.preview;
        if (el.label === 'Start Preview') {
          el.label = 'Stop Preview'
          main()
        } else {
          el.label = 'Start Preview'

          if (fs.existsSync('arrayOfObjects.json')) {
            jsonwriter.end()
            fs.appendFileSync('arrayOfObjects.json', '');
            fs.renameSync('arrayOfObjects.json', 'arrayOfObjects2.json')

            //fs.unlinkSync('log.txt')
          }

          stopMediaTracks(currentStream);
        }
      }
    },

    Startrecord: function () {

      let el = this.$refs.record;
      if (el.label === 'Start Recording') {
        record = true

        if (isElectron) {
          if (!fs.existsSync('arrayOfObjects.json')) {
            logfile = fs.createWriteStream('arrayOfObjects.json')
          }
          jsonwriter.pipe(logfile);
        }
        el.label = 'Stop Recording'
      } else {
        el.label = 'Start Recording'
        record = false
        //  jsonwriter.end()
      }
    },
    Stopmedia() {
      stopMediaTracks(currentStream)
    }
  }
}

async function getCameraList() {
  let cameraList = [];
  await navigator.mediaDevices.getUserMedia({video: true})
  let mediaDevices = await navigator.mediaDevices.enumerateDevices()

  mediaDevices.forEach(mediaDevice => {
    let camera = {}
    if (mediaDevice.kind === 'videoinput') {
      camera.value = mediaDevice.deviceId
      camera.label = mediaDevice.label
      cameraList.push(camera)
    }
  });
  return cameraList
}

// function pause() {
//
//   let x = document.getElementById("pause");
//   if (x.text === "Pause Recording") {
//     x.text = "Paused";
//     currentStream.enabled = false
//   } else {
//     x.text = "Pause Recording";
//     currentStream.enabled = true
//   }
// }

function stopMediaTracks(stream) {

  stream.getTracks().forEach(track => {
    track.stop();
  });
  ctx.clearRect(0, 0, video.width, canvas.height);
  video.height = 0
  canvas.height = 0
  video.srcObject = null
  video = null


}

async function main() {

  await setupCamera();
  video.play();
  videoWidth = video.videoWidth;
  videoHeight = video.videoHeight;
  video.width = videoWidth;
  video.height = videoHeight;
  canvas = document.getElementById('output');
  canvas.width = videoWidth;
  canvas.height = videoHeight;

  ctx = canvas.getContext('2d');
  ctx.translate(canvas.width, 0);
  ctx.scale(-1, 1);
  ctx.fillStyle = '#32EEDB';
  ctx.strokeStyle = '#32EEDB';
  ctx.lineWidth = 0.5;

  model = await facemesh.load({maxFaces: 1});
  await renderPrediction();
}

async function cameraChange() {

  if (typeof currentStream !== 'undefined') {
    stopMediaTracks(currentStream);
    await setupCamera();
  }
}

async function setupCamera() {

  video = document.getElementById('video');

  const stream = await navigator.mediaDevices.getUserMedia({
    'audio': false,
    'video': {
      deviceId: {exact: cameraId},
      facingMode: 'user',
      // Only setting the video to a specified size in order to accommodate a
      // point cloud, so on mobile devices accept the default size.
      width: VIDEO_SIZE,
      height: VIDEO_SIZE
    },
  });
  currentStream = stream;
  video.srcObject = stream;

  return new Promise((resolve) => {
    video.onloadedmetadata = () => {
      resolve(video);
    };
  });
}


function drawPath(ctx, points, closePath) {

  const region = new Path2D();
  region.moveTo(points[0][0], points[0][1]);
  for (let i = 1; i < points.length; i++) {
    const point = points[i];
    region.lineTo(point[0], point[1]);
  }

  if (closePath) {
    region.closePath();
  }
  ctx.stroke(region);
}

async function renderPrediction() {
  const predictions = await model.estimateFaces(video);

  ctx.drawImage(
    video, 0, 0, videoWidth, videoHeight, 0, 0, canvas.width, canvas.height);
  let obj = {}

  if (predictions.length > 0) {
    predictions.forEach(prediction => {
      const keypoints = prediction.scaledMesh;


      if (Triangulationmesh === true) {
        for (let i = 0; i < TRIANGULATION.length / 3; i++) {
          const points = [
            TRIANGULATION[i * 3], TRIANGULATION[i * 3 + 1],
            TRIANGULATION[i * 3 + 2]
          ].map(index => keypoints[index]);

          drawPath(ctx, points, true);
        }
      } else {
        for (let i = 0; i < keypoints.length; i++) {
          const x = keypoints[i][0];
          const y = keypoints[i][1];

          ctx.beginPath();
          ctx.arc(x, y, 1 /* radius */, 0, 2 * Math.PI);
          ctx.fill();
        }
      }
      if (record === true) {
        // output data
        if (isElectron) {
          frameClount += 1;
          obj.faceInViewConfidence = prediction.faceInViewConfidence
          obj.frame = frameClount
          obj.timestamp = Date.now()
          obj.keypoints = keypoints
          console.log(obj)
          jsonwriter.write(obj);
          //let line = JSON.stringify(obj)
          //logfile.write(line + ',');
          // console.log(predictions)
        }
      }
    });
  }

  requestAnimationFrame(renderPrediction);
}

</script>
