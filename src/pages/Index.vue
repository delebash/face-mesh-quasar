<template>
  <div>
      <q-footer>


        <q-tabs v-model="tab">
          <q-tab name="images" label="Images" />
          <q-tab name="videos" label="Videos" />
          <q-tab name="articles" label="Articles" />
        </q-tabs>
      </q-footer>

      <q-page-container class="no-padding">
        <q-page class="row">
          <q-tab-panels class="q-mt-md" v-model="tab" swipeable animated style="width: 100%">

            <q-tab-panel name="images" style="background:red;">
              <div class="text-h6 q-ma-md">Images</div>
              <p class="q-ma-md">Lorem ipsum dolor sit amet consectetur adipisicing elit.</p>
              <div
                class="q-ma-md q-pa-md"
                style="background:black;color:white;width:40%;height:30%;"
              >
                An image which wants to use the a percentage of the page
              </div>
            </q-tab-panel>

            <q-tab-panel swipeable name="videos" style="background:green;">
              <div class="text-h6 q-ma-md">Videos</div>
              <p class="q-ma-md">Lorem ipsum dolor sit amet consectetur adipisicing elit.</p>
            </q-tab-panel>

            <q-tab-panel swipeable name="articles" style="background:blue;">
              <div class="text-h6 q-ma-md">Articles</div>
              <p class="q-ma-md">Lorem ipsum dolor sit amet consectetur adipisicing elit.</p>
            </q-tab-panel>

          </q-tab-panels>
        </q-page>
      </q-page-container>
  </div>
</template>
<script>
export default {
  data () {
    return {
      tab: 'images'
    }
  }
}
</script>
