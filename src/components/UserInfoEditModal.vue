<template>
  <a-modal
    v-model:open="visible"
    title="修改个人信息"
    :confirm-loading="loading"
    @ok="handleSubmit"
    @cancel="handleCancel"
    width="500px"
  >
    <a-form :model="formData" :label-col="{ span: 6 }" :wrapper-col="{ span: 18 }">
      <a-form-item label="用户昵称" name="userName">
        <a-input v-model:value="formData.userName" placeholder="请输入用户昵称" />
      </a-form-item>
      <a-form-item label="用户头像" name="userAvatar">
        <div class="avatar-upload-container">
          <a-upload
            :before-upload="beforeUpload"
            :custom-request="handleUpload"
            :show-upload-list="false"
            accept="image/*"
          >
            <div class="avatar-upload-wrapper">
              <a-avatar
                :src="previewAvatarUrl || uploadedCosUrl || formData.userAvatar"
                :size="80"
                class="avatar-preview"
              >
                {{ formData.userName?.charAt(0) || 'U' }}
              </a-avatar>
              <div class="avatar-upload-mask">
                <CameraOutlined />
                <div>点击上传</div>
              </div>
            </div>
          </a-upload>
          <div v-if="uploading" class="upload-tip">上传中...</div>
        </div>
      </a-form-item>
      <a-form-item label="用户简介" name="userProfile">
        <a-textarea
          v-model:value="formData.userProfile"
          placeholder="请输入用户简介"
          :rows="4"
          :maxlength="500"
          show-count
        />
      </a-form-item>
      <a-divider />
      <a-form-item label="新密码" name="userPassword">
        <a-input-password
          v-model:value="formData.userPassword"
          placeholder="请输入新密码（至少8位）"
          allow-clear
        />
      </a-form-item>
      <a-form-item label="确认密码" name="checkPassword">
        <a-input-password
          v-model:value="formData.checkPassword"
          placeholder="请再次输入新密码"
          allow-clear
        />
      </a-form-item>
      <a-form-item>
        <div class="password-tip">提示：不修改密码请留空</div>
      </a-form-item>
    </a-form>
  </a-modal>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { message } from 'ant-design-vue'
import { CameraOutlined } from '@ant-design/icons-vue'
import type { UploadRequestOption } from 'rc-upload/es/interface'
import { updateMyInfo } from '@/api/userController.ts'
import { useLoginUserStore } from '@/stores/loginUser.ts'
import request from '@/request'

interface Props {
  open: boolean
}

interface Emits {
  (e: 'update:open', value: boolean): void
  (e: 'success'): void
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()
const loginUserStore = useLoginUserStore()

const visible = ref(props.open)
const loading = ref(false)
const uploading = ref(false)
const previewAvatarUrl = ref<string>('')
const uploadedCosUrl = ref<string>('')

const formData = ref<API.UserUpdateMyInfoRequest>({
  userName: '',
  userAvatar: '',
  userProfile: '',
  userPassword: '',
  checkPassword: '',
})

// 监听open变化
watch(
  () => props.open,
  (newVal) => {
    visible.value = newVal
    if (newVal) {
      // 打开时初始化表单数据
      formData.value = {
        userName: loginUserStore.loginUser.userName || '',
        userAvatar: loginUserStore.loginUser.userAvatar || '',
        userProfile: loginUserStore.loginUser.userProfile || '',
        userPassword: '',
        checkPassword: '',
      }
      uploadedCosUrl.value = loginUserStore.loginUser.userAvatar || ''
      previewAvatarUrl.value = ''
    }
  }
)

// 监听visible变化
watch(visible, (newVal) => {
  emit('update:open', newVal)
})

// 上传前校验
const beforeUpload = (file: File) => {
  const isImage = file.type.startsWith('image/')
  if (!isImage) {
    message.error('只能上传图片文件！')
    return false
  }
  const isLt5M = file.size / 1024 / 1024 < 5
  if (!isLt5M) {
    message.error('图片大小不能超过5MB！')
    return false
  }
  // 创建本地预览URL
  const reader = new FileReader()
  reader.onload = (e) => {
    previewAvatarUrl.value = e.target?.result as string
  }
  reader.readAsDataURL(file)
  return false // 阻止自动上传，使用自定义上传
}

// 自定义上传
const handleUpload = async (options: UploadRequestOption) => {
  const { file } = options
  if (!file) {
    return
  }
  uploading.value = true
  try {
    const uploadFormData = new FormData()
    uploadFormData.append('file', file as File)

    const res = await request<API.BaseResponseString>('/file/upload/avatar', {
      method: 'POST',
      data: uploadFormData,
    })

    if (res.data.code === 0 && res.data.data) {
      const cosUrl = res.data.data
      if (cosUrl && cosUrl.trim() !== '') {
        formData.value.userAvatar = cosUrl
        uploadedCosUrl.value = cosUrl
        message.success('头像上传成功')
      } else {
        message.warning('头像上传成功，但未获取到URL')
      }
      previewAvatarUrl.value = ''
    } else {
      message.error('头像上传失败：' + (res.data?.message || '未知错误'))
      previewAvatarUrl.value = ''
    }
  } catch (error: any) {
    const errorMessage = error?.response?.data?.message || error?.message || '请重试'
    message.error('头像上传失败：' + errorMessage)
    previewAvatarUrl.value = ''
  } finally {
    uploading.value = false
  }
}

// 提交表单
const handleSubmit = async () => {
  // 校验昵称
  if (!formData.value.userName?.trim()) {
    message.error('用户昵称不能为空')
    return
  }
  // 校验密码（如果填写了密码）
  if (formData.value.userPassword || formData.value.checkPassword) {
    if (!formData.value.userPassword || formData.value.userPassword.length < 8) {
      message.error('密码长度至少8位')
      return
    }
    if (formData.value.userPassword !== formData.value.checkPassword) {
      message.error('两次输入的密码不一致')
      return
    }
  }
  loading.value = true
  try {
    const finalAvatar = uploadedCosUrl.value || formData.value.userAvatar || ''
    const submitData: API.UserUpdateMyInfoRequest = {
      userName: formData.value.userName || '',
      userAvatar: finalAvatar,
      userProfile: formData.value.userProfile || '',
    }
    // 只有填写了密码才传递密码字段
    if (formData.value.userPassword) {
      submitData.userPassword = formData.value.userPassword
      submitData.checkPassword = formData.value.checkPassword
    }
    const res = await updateMyInfo(submitData)
    if (res.data.code === 0) {
      message.success('修改成功')
      await loginUserStore.fetchLoginUser()
      emit('success')
      visible.value = false
    } else {
      message.error('修改失败：' + res.data.message)
    }
  } catch (error: any) {
    message.error('修改失败：' + (error?.response?.data?.message || error?.message || '请重试'))
  } finally {
    loading.value = false
  }
}

// 取消
const handleCancel = () => {
  visible.value = false
}
</script>

<style scoped>
.ant-form-item {
  margin-bottom: 20px;
}

.avatar-upload-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.avatar-upload-wrapper {
  position: relative;
  cursor: pointer;
  display: inline-block;
}

.avatar-upload-wrapper:hover .avatar-upload-mask {
  opacity: 1;
}

.avatar-preview {
  border: 2px solid rgba(0, 240, 255, 0.5);
  transition: all 0.3s;
  box-shadow: 0 0 10px rgba(0, 240, 255, 0.3);
}

.avatar-upload-mask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #00f0ff;
  opacity: 0;
  transition: opacity 0.3s;
  font-size: 12px;
}

.avatar-upload-mask .anticon {
  font-size: 20px;
  margin-bottom: 4px;
  color: #00f0ff;
}

.upload-tip {
  color: #00d4ff;
  font-size: 12px;
}

.password-tip {
  color: rgba(0, 212, 255, 0.7);
  font-size: 12px;
  margin-top: -16px;
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

:deep(.ant-input),
:deep(.ant-input-password),
:deep(.ant-textarea) {
  background: rgba(0, 0, 0, 0.6) !important;
  border-color: rgba(0, 240, 255, 0.3) !important;
  color: #ffffff !important;
}

:deep(.ant-input:focus),
:deep(.ant-input-focused),
:deep(.ant-input-password:focus),
:deep(.ant-textarea:focus) {
  border-color: #00f0ff !important;
  box-shadow: 0 0 10px rgba(0, 240, 255, 0.3) !important;
}

:deep(.ant-input::placeholder),
:deep(.ant-textarea::placeholder) {
  color: rgba(0, 212, 255, 0.5) !important;
}

:deep(.ant-form-item-label > label) {
  color: #00d4ff;
}

:deep(.ant-divider) {
  border-color: rgba(0, 240, 255, 0.3);
}
</style>

