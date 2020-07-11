<template>
  <q-page>
    Settings
    <q-select filled v-model="model" :options="options" label="Standard"/>
  </q-page>
</template>

<script>
export default {

  data() {
    return {
      model: null,
      options: []
    };
  },
  // App root methods
  async mounted() {
    this.options = await getCameraList()
    console.log(this.options)
  }
};

async function getCameraList() {
  let cameraList = [];
  //await navigator.mediaDevices.getUserMedia({video: true})
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
</script>

<style scoped>

</style>
