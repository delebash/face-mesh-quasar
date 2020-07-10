<template>
  <div>
    <q-footer>
      <q-tabs v-model="tab">
        <q-tab name="preview" label="Preview"/>
        <q-tab name="settings" label="Settings"/>
      </q-tabs>
    </q-footer>

    <q-page-container class="no-padding">
      <q-page class="row">
        <q-tab-panels class="" v-model="tab" swipeable animated style="width: 100%">
          <q-tab-panel class="no-padding" name="preview" style="background:red;">
            <div class="">
              Preview
            </div>
          </q-tab-panel>

          <q-tab-panel class="no-padding" swipeable name="settings"
                       style="background:green;">
            <div class="">
              <q-select v-model="model" :options="options" label="Standard"/>
            </div>
          </q-tab-panel>
        </q-tab-panels>
      </q-page>
    </q-page-container>
  </div>
</template>
<script>
export default {
  data() {
    return {
      tab: '',
      model: null,
      options: []
    }
  },
  async mounted() {
    this.options = await getCameraList()
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
</script>
