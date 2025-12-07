<template>
  <div class="user-info">
    <a-avatar
      :src="user?.userAvatar"
      :size="size"
      :alt="user?.userName || '用户'"
      @error="handleAvatarError"
    >
      {{ user?.userName?.charAt(0) || 'U' }}
    </a-avatar>
    <span v-if="showName" class="user-name">{{ user?.userName || '未知用户' }}</span>
  </div>
</template>

<script setup lang="ts">
interface Props {
  user?: API.UserVO
  size?: number | 'small' | 'default' | 'large'
  showName?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  size: 'default',
  showName: true,
})

// 头像加载失败处理
const handleAvatarError = (e: Event) => {
  // 头像加载失败时，隐藏图片，显示默认头像（文字）
  const target = e.target as HTMLImageElement
  if (target) {
    target.style.display = 'none'
  }
}
</script>

<style scoped>
.user-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.user-name {
  font-size: 14px;
  color: #1a1a1a;
}
</style>
