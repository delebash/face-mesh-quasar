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
            <div class="text-h6">Preview</div>
            <div class="">
              Preview
            </div>
          </q-tab-panel>

          <q-tab-panel class="no-padding" swipeable name="settings"
                       style="background:green;">
            <div class="text-h6">Settings</div>
            <div class="">
              Settings
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
      tab: ''
    }
  }
}
</script>
