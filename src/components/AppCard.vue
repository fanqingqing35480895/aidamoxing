<template>
  <div class="app-card" :class="{ 'app-card--featured': featured }">
    <div class="app-preview">
      <img v-if="app.cover" :src="app.cover" :alt="app.appName" />
      <div v-else class="app-placeholder">
        <span>😈</span>
      </div>
      <div class="app-overlay">
        <a-space>
          <a-button type="primary" @click="handleViewChat">查看对话</a-button>
          <a-button v-if="app.deployKey" type="default" @click="handleViewWork">查看作品</a-button>
        </a-space>
      </div>
    </div>
    <div class="app-info">
      <div class="app-info-left">
        <a-avatar :src="app.user?.userAvatar" :size="40">
          {{ app.user?.userName?.charAt(0) || 'U' }}
        </a-avatar>
      </div>
      <div class="app-info-right">
        <h3 class="app-title">{{ app.appName || '未命名应用' }}</h3>
        <p class="app-author">
          {{ app.user?.userName || (featured ? '官方' : '未知用户') }}
        </p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
interface Props {
  app: API.AppVO
  featured?: boolean
}

interface Emits {
  (e: 'view-chat', appId: string | number | undefined): void
  (e: 'view-work', app: API.AppVO): void
}

const props = withDefaults(defineProps<Props>(), {
  featured: false,
})

const emit = defineEmits<Emits>()

const handleViewChat = () => {
  emit('view-chat', props.app.id)
}

const handleViewWork = () => {
  emit('view-work', props.app)
}
</script>

<style scoped>
.app-card {
  background: rgba(0, 0, 0, 0.6);
  border-radius: 16px;
  overflow: hidden;
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.5),
    0 0 20px rgba(0, 240, 255, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(0, 240, 255, 0.3);
  transition: all 0.3s;
  cursor: pointer;
}

.app-card:hover {
  transform: translateY(-8px);
  box-shadow:
    0 15px 50px rgba(0, 0, 0, 0.7),
    0 0 40px rgba(0, 240, 255, 0.4);
  border-color: #00f0ff;
}

.app-card--featured {
  border-color: rgba(0, 240, 255, 0.5);
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.5),
    0 0 30px rgba(0, 240, 255, 0.2);
}

.app-preview {
  height: 180px;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  position: relative;
}

.app-preview img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.9;
}

.app-placeholder {
  font-size: 48px;
  color: rgba(0, 212, 255, 0.3);
  filter: drop-shadow(0 0 10px rgba(0, 240, 255, 0.5));
}

.app-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s;
  backdrop-filter: blur(5px);
}

.app-card:hover .app-overlay {
  opacity: 1;
}

.app-overlay :deep(.ant-btn-primary) {
  background: rgba(0, 240, 255, 0.2);
  border-color: #00f0ff;
  color: #00f0ff;
  box-shadow: 0 0 15px rgba(0, 240, 255, 0.3);
}

.app-overlay :deep(.ant-btn-primary:hover) {
  background: rgba(0, 240, 255, 0.3);
  box-shadow: 0 0 25px rgba(0, 240, 255, 0.5);
}

.app-overlay :deep(.ant-btn-default) {
  background: rgba(0, 0, 0, 0.6);
  border-color: rgba(0, 240, 255, 0.5);
  color: #00d4ff;
}

.app-overlay :deep(.ant-btn-default:hover) {
  border-color: #00f0ff;
  color: #00f0ff;
  box-shadow: 0 0 15px rgba(0, 240, 255, 0.3);
}

.app-info {
  padding: 16px;
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(0, 0, 0, 0.4);
}

.app-info-left :deep(.ant-avatar) {
  border: 2px solid rgba(0, 240, 255, 0.3);
  box-shadow: 0 0 10px rgba(0, 240, 255, 0.2);
}

.app-info-right {
  flex: 1;
  min-width: 0;
}

.app-title {
  font-size: 16px;
  font-weight: 600;
  margin: 0 0 4px;
  color: #00f0ff;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  text-shadow: 0 0 10px rgba(0, 240, 255, 0.5);
}

.app-author {
  font-size: 14px;
  color: rgba(0, 212, 255, 0.7);
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
