<template>
  <q-layout view="hHh lpR lfr">

    <q-header elevated class="bg-primary text-white">
      <Mainmenu></Mainmenu>
    </q-header>

    <q-page-container>
      <router-view/>
    </q-page-container>

  </q-layout>
</template>

<script>
import Mainmenu from "../components/MainMenu";

export default {
  components: {Mainmenu},
  component: {
    'mainmenu': require('../components/MainMenu').default
  }
}

</script>
