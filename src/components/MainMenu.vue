<template>
  <q-toolbar>
    <q-toolbar-title>
      <a href="/" style="text-decoration:none;color:white;visited:white">
        <q-avatar>
          <img src="https://cdn.quasar.dev/logo/svg/quasar-logo.svg">
        </q-avatar>
        Face Mesh
      </a>
    </q-toolbar-title>

    <q-tabs items-left active-bg-color="red" indicator-color="white">
      <q-route-tab
        label="View Face"
        to="face"
        exact
      />
      <q-route-tab
        label="Settings"
        to="settings"
        exact
      />
    </q-tabs>
  </q-toolbar>


</template>

<script>
export default {}
</script>
