<script lang="ts" setup>
import { computed } from 'vue'
import { useAppStore } from '@/stores/app'
import Navbar from './components/Navbar.vue'
import Sidebar from './components/Sidebar/index.vue'
import AppMain from './components/AppMain.vue'
import useResizeHandler from './composables/useResizeHandler'

const appStore = useAppStore()

const sidebar = computed(() => appStore.sidebar)
const device = computed(() => appStore.device)

const classObj = computed(() => ({
  hideSidebar: !sidebar.value.opened,
  withoutAnimation: sidebar.value.withoutAnimation,
  mobile: device.value === 'mobile'
}))

useResizeHandler()
</script>

<template>
  <div class="app-wrapper" :class="classObj">
    <Sidebar class="sidebar-container" />
    <div class="main-container">
      <Navbar />
      <AppMain />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.app-wrapper {
  position: relative;
  height: 100%;
  width: 100%;
}
</style>
