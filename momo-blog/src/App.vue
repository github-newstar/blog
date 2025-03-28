<template>
  <div>
    <NavBar />
    <div class="shadow-md h-fit w-4/5 mx-auto">
      <router-view />
    </div>
  </div>
</template>

<script setup>
import NavBar from './components/NavBar.vue'
import { usePageStateStore } from './stores/PageState'


</script>

<style lang="scss" scoped></style>
