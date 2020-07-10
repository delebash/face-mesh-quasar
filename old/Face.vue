<template>
  <q-page padding>
    View Face

    <div class="container">
      <div>
        <canvas id="output"></canvas>
        <video id="video"></video>
      </div>
    </div>
  </q-page>
</template>

<script>


let model, ctx, videoWidth, videoHeight, video, canvas, currentStream,
  scatterGLHasInitialized = false, scatterGL, triangulateMesh = false;
const VIDEO_SIZE = 500;
import '@tensorflow/tfjs'
// import '@tensorflow/tfjs-node-gpu';
import * as facemesh from '@tensorflow-models/facemesh'

import TRIANGULATION from '../js/triangulation'

export default {
  data() {
    return {
      model: null,
      options: []
    }
  },
  async mounted() {
    await getInputDevices()
  },
  methods: {
    startpreview: function () {
      main()
    },
    record: function () {
      //main()
    },
    stopMedia() {
      stopMediaTracks(currentStream)
    },
    pauseMedia() {
      //  pause()
    }
  }
}

function pause() {

  let x = document.getElementById("pause");
  if (x.text === "Pause Recording") {
    x.text = "Paused";
    currentStream.enabled = false
  } else {
    x.text = "Pause Recording";
    currentStream.enabled = true
  }
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
  video.srcObject = null
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  video = null
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

  }
  requestAnimationFrame(renderPrediction);
}
</script>

<style scoped>

</style>
