<template>
  <q-layout view="hHh lpR lfr">

    <q-header elevated class="bg-primary text-white">
      <TitleBar></TitleBar>
    </q-header>

    <q-page-container>
      <router-view/>
    </q-page-container>

  </q-layout>
</template>

<script>

import TitleBar from "../components/TitleBar";

export default {
  components: {TitleBar},
}

</script>
