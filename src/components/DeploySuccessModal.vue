<template>
  <a-modal v-model:open="visible" title="部署成功" :footer="null" width="600px">
    <div class="deploy-success">
      <div class="success-icon">
        <CheckCircleOutlined style="color: #52c41a; font-size: 48px" />
      </div>
      <h3>网站部署成功！</h3>
      <p>你的网站已经成功部署，可以通过以下链接访问：</p>
      <div class="deploy-url">
        <a-input :value="deployUrl" readonly>
          <template #suffix>
            <a-button type="text" @click="handleCopyUrl">
              <CopyOutlined />
            </a-button>
          </template>
        </a-input>
      </div>
      <div class="deploy-actions">
        <a-button type="primary" @click="handleOpenSite">访问网站</a-button>
        <a-button @click="handleClose">关闭</a-button>
      </div>
    </div>
  </a-modal>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { message } from 'ant-design-vue'
import { CheckCircleOutlined, CopyOutlined } from '@ant-design/icons-vue'

interface Props {
  open: boolean
  deployUrl: string
}

interface Emits {
  (e: 'update:open', value: boolean): void
  (e: 'open-site'): void
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()

const visible = computed({
  get: () => props.open,
  set: (value) => emit('update:open', value),
})

const handleCopyUrl = async () => {
  try {
    await navigator.clipboard.writeText(props.deployUrl)
    message.success('链接已复制到剪贴板')
  } catch (error) {
    console.error('复制失败：', error)
    message.error('复制失败')
  }
}

const handleOpenSite = () => {
  emit('open-site')
}

const handleClose = () => {
  visible.value = false
}
</script>

<style scoped>
.deploy-success {
  text-align: center;
  padding: 24px;
  color: #ffffff;
}

.success-icon {
  margin-bottom: 16px;
}

.success-icon :deep(.anticon) {
  color: #00f0ff !important;
  filter: drop-shadow(0 0 10px rgba(0, 240, 255, 0.8));
}

.deploy-success h3 {
  margin: 0 0 16px;
  font-size: 20px;
  font-weight: 600;
  color: #00f0ff;
  text-shadow: 0 0 10px rgba(0, 240, 255, 0.5);
}

.deploy-success p {
  margin: 0 0 24px;
  color: rgba(0, 212, 255, 0.7);
}

.deploy-url {
  margin-bottom: 24px;
}

.deploy-url :deep(.ant-input) {
  background: rgba(0, 0, 0, 0.6) !important;
  border-color: rgba(0, 240, 255, 0.3) !important;
  color: #ffffff !important;
}

.deploy-actions {
  display: flex;
  gap: 12px;
  justify-content: center;
}

:deep(.ant-modal-content) {
  background: rgba(0, 0, 0, 0.9);
  border: 1px solid rgba(0, 240, 255, 0.3);
}

:deep(.ant-modal-header) {
  background: rgba(0, 0, 0, 0.9);
  border-bottom-color: rgba(0, 240, 255, 0.3);
}

:deep(.ant-modal-title) {
  color: #00f0ff;
}

:deep(.ant-modal-body) {
  color: #ffffff;
}

:deep(.ant-btn-primary) {
  background: rgba(0, 240, 255, 0.2);
  border-color: #00f0ff;
  color: #00f0ff;
  box-shadow: 0 0 15px rgba(0, 240, 255, 0.3);
}

:deep(.ant-btn-primary:hover) {
  background: rgba(0, 240, 255, 0.3);
  box-shadow: 0 0 25px rgba(0, 240, 255, 0.5);
}

:deep(.ant-btn-default) {
  background: rgba(0, 0, 0, 0.6);
  border-color: rgba(0, 240, 255, 0.3);
  color: #00d4ff;
}

:deep(.ant-btn-default:hover) {
  border-color: #00f0ff;
  color: #00f0ff;
  box-shadow: 0 0 15px rgba(0, 240, 255, 0.3);
}

:deep(.ant-btn-text) {
  color: #00d4ff;
}

:deep(.ant-btn-text:hover) {
  color: #00f0ff;
}
</style>
