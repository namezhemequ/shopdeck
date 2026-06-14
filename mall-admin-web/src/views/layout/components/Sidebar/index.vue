<script lang="ts" setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import SidebarItem from './SidebarItem.vue'
import ScrollBar from '@/components/ScrollBar/index.vue'
import { useAppStore } from '@/stores/app'
import usePermissionStore from '@/stores/permission'

defineOptions({
  name: 'Sidebar'
})

const appStore = useAppStore()
const permissionStore = usePermissionStore()
const route = useRoute()

const routes = computed(() => permissionStore.routers)
const isCollapse = computed(() => !appStore.sidebar.opened)
</script>

<template>
  <div class="sidebar-container">
    <!-- Brand Logo -->
    <div class="sidebar-logo">
      <div class="sidebar-logo-icon">SD</div>
      <transition name="fade">
        <span v-show="!isCollapse" class="sidebar-logo-text">ShopDeck</span>
      </transition>
    </div>
    <!-- Menu -->
    <scroll-bar>
      <el-menu
        mode="vertical"
        :show-timeout="200"
        :default-active="route.path"
        :collapse="isCollapse"
        class="sidebar-menu"
      >
        <sidebar-item :routes="routes" />
      </el-menu>
    </scroll-bar>
  </div>
</template>

<style lang="scss" scoped>
.sidebar-logo {
  display: flex;
  align-items: center;
  height: 56px;
  padding: 0 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  overflow: hidden;
}

.sidebar-logo-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  min-width: 32px;
  border-radius: 8px;
  background: linear-gradient(135deg, #4F46E5, #6366F1);
  color: #fff;
  font-size: 14px;
  font-weight: 700;
  letter-spacing: -0.5px;
}

.sidebar-logo-text {
  margin-left: 12px;
  color: #fff;
  font-size: 17px;
  font-weight: 600;
  white-space: nowrap;
  letter-spacing: -0.3px;
}

.sidebar-menu {
  border-right: none !important;
}
</style>
