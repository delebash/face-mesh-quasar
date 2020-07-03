<template>
  <q-page padding>
    View Face
    <q-select filled v-model="model" :options="options" label="Standard"/>
    <div class="container">
      <div class="canvas-wrapper">
        <canvas id="output"></canvas>
        <video id="video" playsinline style="
          -webkit-transform: scaleX(-1);
          transform: scaleX(-1);
          visibility: hidden;
          width: auto;
          height: auto;
          ">
        </video>
      </div>
      <div id="scatter-gl-container"></div>
    </div>
  </q-page>
</template>

<script>
let model, ctx, videoWidth, videoHeight, video, canvas, currentStream,
  scatterGLHasInitialized = false, scatterGL, triangulateMesh = true;
const VIDEO_SIZE = 500;
import * as facemesh from '@tensorflow-models/facemesh'

import TRIANGULATION from './triangulation'

// import '@tensorflow/tfjs-node-gpu';
let cameras
export default {
  data() {
    return {
      model: null,
      options: options
    }
  },
  mounted() {
    console.log('face mounted')
    main()
  }
}

async function main() {

   this.options = await getInputDevices()
  console.log('face mounted')

}

async function getInputDevices() {
  await navigator.mediaDevices.getUserMedia({video: true})
  let videoDevices = await navigator.mediaDevices.enumerateDevices()
  let devices = []
  let obj = {}
  videoDevices.forEach((element, index, array) => {
      if (element.kind === 'videoinput') {
        obj.label = element.label
        obj.value = element.deviceId
        obj.description = ""
        obj.category = ""
        devices.push(obj)
        // console.log(element.deviceId); // 100, 200, 300
        // console.log(index); // 0, 1, 2
        // console.log(array); // same myArray object 3 times
      }
    }
  );
  return devices
}

async function cameraChange(cameraId) {
  if (typeof currentStream !== 'undefined') {
    stopMediaTracks(currentStream);
    await setupCamera(cameraId);
  }
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


function stopMediaTracks(stream) {
  stream.getTracks().forEach(track => {
    track.stop();
  });
}


async function setupCamera(cameraId) {
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

async function renderPrediction() {

  const predictions = await model.estimateFaces(video);
  ctx.drawImage(
    video, 0, 0, videoWidth, videoHeight, 0, 0, canvas.width, canvas.height);

  if (predictions.length > 0) {
    predictions.forEach(prediction => {
      const keypoints = prediction.scaledMesh;

      if (triangulateMesh === true) {
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
    });

    if (scatterGL != null) {
      const pointsData = predictions.map(prediction => {
        let scaledMesh = prediction.scaledMesh;
        return scaledMesh.map(point => ([-point[0], -point[1], -point[2]]));
      });

      let flattenedPointsData = [];
      for (let i = 0; i < pointsData.length; i++) {
        flattenedPointsData = flattenedPointsData.concat(pointsData[i]);
      }
      const dataset = new ScatterGL.Dataset(flattenedPointsData);

      if (!scatterGLHasInitialized) {
        scatterGL.render(dataset);
      } else {
        scatterGL.updateDataset(dataset);
      }
      scatterGLHasInitialized = true;
    }
  }
  requestAnimationFrame(renderPrediction);
}


//document.getElementById('main').appendChild(stats.dom);

// await setupCamera(cameraId);
// video.play();
// videoWidth = video.videoWidth;
// videoHeight = video.videoHeight;
// video.width = videoWidth;
// video.height = videoHeight;
//
// canvas = document.getElementById('output');
// canvas.width = videoWidth;
// canvas.height = videoHeight;
// const canvasContainer = document.querySelector('.canvas-wrapper');
// canvasContainer.style = `width: ${videoWidth}px; height: ${videoHeight}px`;
//
// ctx = canvas.getContext('2d');
// ctx.translate(canvas.width, 0);
// ctx.scale(-1, 1);
// ctx.fillStyle = '#32eedb';
// ctx.strokeStyle = '#32EEDB';
// ctx.lineWidth = 0.5;
//
// model = await facemesh.load({maxFaces: 2});
// await renderPrediction();

</script>

<style scoped>

</style>
