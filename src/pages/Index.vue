<template>
  <div>
    <div>
        <q-tab-panels v-model="tab" animated>
          <q-tab-panel name="cameras">
            <div class="text-h6">Cameras</div>
            Lorem ipsum dolor sit amet consectetur adipisicing elit.
          </q-tab-panel>

          <q-tab-panel name="settings">
            <div class="text-h6">Settings</div>
            Lorem ipsum dolor sit amet consectetur adipisicing elit.
          </q-tab-panel>
        </q-tab-panels>

        <q-separator/>
      <q-footer>
        <q-tabs
          v-model="tab"
          dense
          class="text-grey"
          active-color="primary"
          indicator-color="primary"
          align="justify"
          narrow-indicator
        >

            <q-tab name="cameras" label="Cameras"/>
            <q-tab name="settings" label="Settings"/>
        </q-tabs>
      </q-footer>

    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tab: ''
    }
  }
}
</script>

<style scoped>

</style>
