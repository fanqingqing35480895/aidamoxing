<template>
  <a-layout-header class="header">
    <a-row :wrap="false">
      <!-- 左侧：Logo和标题 -->
      <a-col flex="200px">
        <RouterLink to="/">
          <div class="header-left">
            <img class="logo" src="@/assets/logo.png" alt="Logo" />
            <h1 class="site-title">ai生成</h1>
          </div>
        </RouterLink>
      </a-col>
      <!-- 中间：导航菜单 -->
      <a-col flex="auto">
        <a-menu
          v-model:selectedKeys="selectedKeys"
          mode="horizontal"
          :items="menuItems"
          @click="handleMenuClick"
        />
      </a-col>
      <!-- 右侧：用户操作区域 -->
      <a-col>
        <div class="user-login-status">
          <div v-if="loginUserStore.loginUser.id">
            <a-dropdown>
              <a-space>
                <a-avatar
                  :src="loginUserStore.loginUser.userAvatar"
                  @error="handleAvatarError"
                >
                  {{ loginUserStore.loginUser.userName?.charAt(0) || 'U' }}
                </a-avatar>
                {{ loginUserStore.loginUser.userName ?? '无名' }}
              </a-space>
              <template #overlay>
                <a-menu>
                  <a-menu-item @click="showEditModal = true">
                    <UserOutlined />
                    修改信息
                  </a-menu-item>
                  <a-menu-item @click="doLogout">
                    <LogoutOutlined />
                    退出登录
                  </a-menu-item>
                </a-menu>
              </template>
            </a-dropdown>
          </div>
          <div v-else>
            <a-button type="primary" href="/user/login">登录</a-button>
          </div>
        </div>
      </a-col>
    </a-row>
    <UserInfoEditModal v-model:open="showEditModal" @success="handleEditSuccess" />
  </a-layout-header>
</template>

<script setup lang="ts">
import { computed, h, ref } from 'vue'
import { useRouter } from 'vue-router'
import { type MenuProps, message } from 'ant-design-vue'
import { useLoginUserStore } from '@/stores/loginUser.ts'
import { userLogout } from '@/api/userController.ts'
import { LogoutOutlined, HomeOutlined, UserOutlined } from '@ant-design/icons-vue'
import UserInfoEditModal from './UserInfoEditModal.vue'

const loginUserStore = useLoginUserStore()
const router = useRouter()
const showEditModal = ref(false)
// 当前选中菜单
const selectedKeys = ref<string[]>(['/'])
// 监听路由变化，更新当前选中菜单
router.afterEach((to, from, next) => {
  selectedKeys.value = [to.path]
})

// 菜单配置项
const originItems = [
  {
    key: '/',
    icon: () => h(HomeOutlined),
    label: '主页',
    title: '主页',
  },
  {
    key: '/admin/userManage',
    label: '用户管理',
    title: '用户管理',
  },
  {
    key: '/admin/appManage',
    label: '应用管理',
    title: '应用管理',
  },
  {
    key: 'others',
    label: h('a', { href: 'https://www.deepseek.com', target: '_blank' }, '优化调用'),
    title: '优化调用',
  },
]

// 过滤菜单项
const filterMenus = (menus = [] as MenuProps['items']) => {
  return menus?.filter((menu) => {
    const menuKey = menu?.key as string
    if (menuKey?.startsWith('/admin')) {
      const loginUser = loginUserStore.loginUser
      if (!loginUser || loginUser.userRole !== 'admin') {
        return false
      }
    }
    return true
  })
}

// 展示在菜单的路由数组
const menuItems = computed<MenuProps['items']>(() => filterMenus(originItems))

// 处理菜单点击
const handleMenuClick: MenuProps['onClick'] = (e) => {
  const key = e.key as string
  selectedKeys.value = [key]
  // 跳转到对应页面
  if (key.startsWith('/')) {
    router.push(key)
  }
}

// 退出登录
const doLogout = async () => {
  const res = await userLogout()
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await router.push('/user/login')
  } else {
    message.error('退出登录失败，' + res.data.message)
  }
}

// 修改信息成功后的回调
const handleEditSuccess = () => {
  // 用户信息已自动更新，无需额外操作
}

// 头像加载失败处理
const handleAvatarError = (e: Event) => {
  const img = e.target as HTMLImageElement
  if (img) {
    img.style.display = 'none'
  }
}
</script>
<style scoped>
.header {
  background: rgba(0, 0, 0, 0.8);
  padding: 0 24px;
  border-bottom: 1px solid rgba(0, 240, 255, 0.3);
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 20px rgba(0, 240, 255, 0.1);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo {
  height: 48px;
  width: 48px;
  filter: drop-shadow(0 0 10px rgba(0, 240, 255, 0.5));
}

.site-title {
  margin: 0;
  font-size: 18px;
  color: #00f0ff;
  text-shadow: 0 0 10px rgba(0, 240, 255, 0.8);
}

:deep(.ant-menu-horizontal) {
  border-bottom: none !important;
  background: transparent !important;
}

:deep(.ant-menu-item) {
  color: rgba(0, 212, 255, 0.8) !important;
}

:deep(.ant-menu-item:hover) {
  color: #00f0ff !important;
  background: rgba(0, 240, 255, 0.1) !important;
}

:deep(.ant-menu-item-selected) {
  color: #00f0ff !important;
  border-bottom-color: #00f0ff !important;
}

:deep(.ant-menu-item-selected::after) {
  border-bottom-color: #00f0ff !important;
}

.user-login-status :deep(.ant-space) {
  color: #00d4ff;
}

.user-login-status :deep(.ant-avatar) {
  border: 2px solid rgba(0, 240, 255, 0.5);
  box-shadow: 0 0 10px rgba(0, 240, 255, 0.3);
}

.user-login-status :deep(.ant-dropdown-menu) {
  background: rgba(0, 0, 0, 0.9) !important;
  border: 1px solid rgba(0, 240, 255, 0.3) !important;
  box-shadow: 0 0 20px rgba(0, 240, 255, 0.2) !important;
}

.user-login-status :deep(.ant-menu-item) {
  color: #00d4ff !important;
}

.user-login-status :deep(.ant-menu-item:hover) {
  background: rgba(0, 240, 255, 0.1) !important;
  color: #00f0ff !important;
}

.user-login-status :deep(.ant-btn-primary) {
  background: rgba(0, 240, 255, 0.2);
  border-color: #00f0ff;
  color: #00f0ff;
  box-shadow: 0 0 15px rgba(0, 240, 255, 0.3);
}

.user-login-status :deep(.ant-btn-primary:hover) {
  background: rgba(0, 240, 255, 0.3);
  border-color: #00f0ff;
  box-shadow: 0 0 25px rgba(0, 240, 255, 0.5);
}
</style>
