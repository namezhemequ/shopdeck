<script lang="ts" setup>
import { computed } from 'vue'
import Breadcrumb from '@/components/Breadcrumb/index.vue'
import Hamburger from '@/components/Hamburger/index.vue'
import { useAppStore } from '@/stores/app'
import { useUserStore } from '@/stores/user'
import { ArrowDown, SwitchButton } from '@element-plus/icons-vue'

defineOptions({
  name: 'Navbar'
})

const appStore = useAppStore()
const userStore = useUserStore()

const sidebar = computed(() => appStore.sidebar)
const avatar = computed(() => userStore.userInfo.avatar)

const handleToggleSideBar = () => {
  appStore.toggleSideBar()
}

const handleLogout = async () => {
  await userStore.userLogout()
  location.reload()
}
</script>

<template>
  <div class="navbar">
    <div class="navbar-left">
      <hamburger
        class="hamburger-container"
        :toggle-click="handleToggleSideBar"
        :is-active="sidebar.opened"
      />
      <breadcrumb />
    </div>
    <div class="navbar-right">
      <el-dropdown class="avatar-container" trigger="click">
        <div class="avatar-wrapper">
          <img class="user-avatar" :src="avatar" />
          <span class="avatar-name">{{ userStore.userInfo.username }}</span>
          <el-icon class="avatar-arrow">
            <arrow-down />
          </el-icon>
        </div>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item>
              <router-link to="/">首页</router-link>
            </el-dropdown-item>
            <el-dropdown-item divided @click="handleLogout">
              <el-icon><SwitchButton /></el-icon>
              <span>退出登录</span>
            </el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@use '@/styles/variables.scss' as v;

.navbar {
  height: v.$navbarHeight;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: v.$bg-card;
  border-bottom: 1px solid v.$border-color;
  box-shadow: v.$shadow-sm;
  padding: 0 16px;
}

.navbar-left {
  display: flex;
  align-items: center;
  height: 100%;
}

.navbar-right {
  display: flex;
  align-items: center;
  height: 100%;
}

.hamburger-container {
  display: flex;
  align-items: center;
  height: 100%;
  padding: 0 8px;
  cursor: pointer;
}

.avatar-container {
  height: 100%;
  display: flex;
  align-items: center;
}

.avatar-wrapper {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: v.$border-radius;
  transition: background v.$transition-fast;

  &:hover {
    background: v.$bg-component;
  }
}

.user-avatar {
  width: 34px;
  height: 34px;
  border-radius: 8px;
  border: 2px solid v.$border-color;
}

.avatar-name {
  font-size: 14px;
  color: v.$text-primary;
  font-weight: 500;
}

.avatar-arrow {
  font-size: 12px;
  color: v.$text-secondary;
}
</style>
