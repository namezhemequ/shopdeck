<script lang="ts" setup>
import { reactive, ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { isvalidUsername } from '@/utils/validate'
import { type FormInstance, type FormRules } from 'element-plus'
import { User, Lock, Loading } from '@element-plus/icons-vue'

const router = useRouter()
const userStore = useUserStore()

const loginFormRef = ref<FormInstance>()

const loginForm = reactive({
  username: '',
  password: '',
})

const loginRules = reactive<FormRules<typeof loginForm>>({
  username: [{ required: true, trigger: 'blur', validator: validateUsername }],
  password: [{ required: true, trigger: 'blur', validator: validatePass }]
})

const loading = ref(false)

function validateUsername(_rule: unknown, value: string, callback: (error?: Error) => void) {
  if (!isvalidUsername(value)) {
    callback(new Error('请输入正确的用户名'))
  } else {
    callback()
  }
}

function validatePass(_rule: unknown, value: string, callback: (error?: Error) => void) {
  if (value.length < 3) {
    callback(new Error('密码不能小于3位'))
  } else {
    callback()
  }
}

onMounted(() => {
  loginForm.username = userStore.userInfo.username
  loginForm.password = userStore.userInfo.password
  if (!loginForm.username) {
    loginForm.username = 'admin'
  }
})

const handleLogin = () => {
  loginFormRef.value!.validate(async (valid) => {
    if (valid) {
      loading.value = true
      try {
        await userStore.userLogin({
          username: loginForm.username.trim(),
          password: loginForm.password
        })
        loading.value = false
        router.push({ path: '/' })
      } catch (err) {
        loading.value = false
        console.log(err)
      }
    }
  })
}
</script>

<template>
  <div class="login-page">
    <!-- Left Brand Panel -->
    <div class="login-brand">
      <div class="brand-content">
        <div class="brand-logo">SD</div>
        <h1 class="brand-name">ShopDeck</h1>
        <p class="brand-tagline">电商后台管理系统</p>
        <div class="brand-features">
          <div class="feature-item">
            <div class="feature-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
            </div>
            <span>商品 · 订单 · 营销一站式管理</span>
          </div>
          <div class="feature-item">
            <div class="feature-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
            </div>
            <span>实时数据监控与可视化分析</span>
          </div>
          <div class="feature-item">
            <div class="feature-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
            </div>
            <span>安全可靠的权限管理体系</span>
          </div>
        </div>
      </div>
      <div class="brand-decoration">
        <div class="deco-circle deco-1" />
        <div class="deco-circle deco-2" />
        <div class="deco-circle deco-3" />
      </div>
    </div>

    <!-- Right Form Panel -->
    <div class="login-form-panel">
      <div class="form-card">
        <div class="form-header">
          <h2>欢迎回来</h2>
          <p>登录您的管理账户</p>
        </div>
        <el-form
          ref="loginFormRef"
          :model="loginForm"
          :rules="loginRules"
          class="login-form"
          @keyup.enter="handleLogin"
        >
          <el-form-item prop="username">
            <el-input
              v-model="loginForm.username"
              placeholder="用户名"
              size="large"
              :prefix-icon="User"
            />
          </el-form-item>
          <el-form-item prop="password">
            <el-input
              v-model="loginForm.password"
              type="password"
              placeholder="密码"
              size="large"
              show-password
              :prefix-icon="Lock"
            />
          </el-form-item>
          <el-form-item>
            <el-button
              type="primary"
              size="large"
              class="login-btn"
              :loading="loading"
              @click="handleLogin"
            >
              <el-icon v-if="loading"><Loading /></el-icon>
              <span>{{ loading ? '登录中...' : '登 录' }}</span>
            </el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@use '@/styles/variables.scss' as v;

.login-page {
  display: flex;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

// ============================================================
// Left Brand Panel
// ============================================================
.login-brand {
  position: relative;
  flex: 0 0 55%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #1E293B 0%, #312E81 40%, #4F46E5 100%);
  overflow: hidden;
}

.brand-content {
  position: relative;
  z-index: 2;
  max-width: 440px;
  padding: 40px;
  animation: fadeInUp 0.8s ease;
}

.brand-logo {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 56px;
  height: 56px;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  color: #fff;
  font-size: 24px;
  font-weight: 700;
  letter-spacing: -1px;
  margin-bottom: 24px;
}

.brand-name {
  color: #fff;
  font-size: 36px;
  font-weight: 700;
  margin: 0 0 8px 0;
  letter-spacing: -1px;
}

.brand-tagline {
  color: rgba(255, 255, 255, 0.7);
  font-size: 16px;
  margin: 0 0 48px 0;
}

.brand-features {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 12px;
  color: rgba(255, 255, 255, 0.75);
  font-size: 15px;
}

.feature-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  flex-shrink: 0;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.1);

  svg {
    width: 16px;
    height: 16px;
    color: rgba(255, 255, 255, 0.8);
  }
}

// Decorative circles
.brand-decoration {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.deco-circle {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.03);
}

.deco-1 {
  width: 500px;
  height: 500px;
  top: -150px;
  right: -100px;
}

.deco-2 {
  width: 300px;
  height: 300px;
  bottom: -80px;
  left: -80px;
}

.deco-3 {
  width: 200px;
  height: 200px;
  top: 50%;
  left: 60%;
}

// ============================================================
// Right Form Panel
// ============================================================
.login-form-panel {
  flex: 0 0 45%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: v.$bg-page;
}

.form-card {
  width: 100%;
  max-width: 400px;
  padding: 48px 40px;
  background: v.$bg-card;
  border-radius: 16px;
  box-shadow: v.$shadow-lg;
  animation: fadeInUp 0.6s ease 0.15s both;
}

.form-header {
  text-align: center;
  margin-bottom: 36px;

  h2 {
    margin: 0 0 8px 0;
    font-size: 26px;
    font-weight: 700;
    color: v.$text-primary;
  }

  p {
    margin: 0;
    font-size: 15px;
    color: v.$text-secondary;
  }
}

.login-form {
  :deep(.el-input--large) {
    --el-input-height: 48px;
    --el-input-border-radius: 10px;
  }

  :deep(.el-form-item) {
    margin-bottom: 20px;
  }
}

.login-btn {
  width: 100%;
  height: 48px;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 500;
  letter-spacing: 2px;
  transition: all v.$transition-fast;

  &:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(79, 70, 229, 0.4);
  }
}

// ============================================================
// Animations
// ============================================================
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(24px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
