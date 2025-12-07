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
                :src="previewAvatarUrl || formData.userAvatar"
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
// 本地预览URL（选择文件后立即显示）
const previewAvatarUrl = ref<string>('')
// 上传成功的COS URL（防止formData重置时丢失）
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
      // 如果有已上传的COS URL，优先使用；否则使用用户原有的头像
      const initialAvatar = uploadedCosUrl.value || loginUserStore.loginUser.userAvatar || ''
      formData.value = {
        userName: loginUserStore.loginUser.userName || '',
        userAvatar: initialAvatar,
        userProfile: loginUserStore.loginUser.userProfile || '',
        userPassword: '',
        checkPassword: '',
      }
      // 重置预览URL（但保留已上传的COS URL）
      previewAvatarUrl.value = ''
    } else {
      // 关闭模态框时，清空已上传的COS URL（如果用户没有提交）
      uploadedCosUrl.value = ''
    }
  }
)

// 监听visible变化
watch(visible, (newVal) => {
  emit('update:open', newVal)
})

// 上传前校验并显示预览
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
      // 不手动设置Content-Type，让axios自动设置（包含boundary）
    })
    if (res.data.code === 0 && res.data.data) {
      // 保存COS存储地址
      const cosUrl = res.data.data
      if (cosUrl) {
        // 同时保存到formData和uploadedCosUrl，确保不会丢失
        formData.value.userAvatar = cosUrl
        uploadedCosUrl.value = cosUrl
        console.log('头像上传成功，COS地址已保存：', cosUrl)
        console.log('formData.userAvatar:', formData.value.userAvatar)
        console.log('uploadedCosUrl:', uploadedCosUrl.value)
      } else {
        console.error('头像上传成功，但返回的URL为空')
        message.warning('头像上传成功，但未获取到URL')
      }
      // 上传成功后，使用服务器返回的URL，清除本地预览
      previewAvatarUrl.value = ''
      message.success('头像上传成功')
    } else {
      message.error('头像上传失败：' + (res.data.message || '未知错误'))
      // 上传失败，清除预览
      previewAvatarUrl.value = ''
    }
  } catch (error: any) {
    console.error('头像上传错误：', error)
    const errorMessage = error?.response?.data?.message || error?.message || '请重试'
    message.error('头像上传失败：' + errorMessage)
    // 上传失败，清除预览
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
    // 构建提交数据 - 确保所有字段都被传递
    // 优先使用已上传的COS URL，如果没有则使用formData中的值
    const finalAvatar = uploadedCosUrl.value || formData.value.userAvatar || ''
    const submitData: API.UserUpdateMyInfoRequest = {
      userName: formData.value.userName || '',
      userAvatar: finalAvatar, // COS存储地址，优先使用已上传的URL
      userProfile: formData.value.userProfile || '',
    }
    // 记录提交的数据，用于调试
    console.log('=== 提交用户信息 ===')
    console.log('userName:', submitData.userName)
    console.log('formData.userAvatar:', formData.value.userAvatar || '（空）')
    console.log('uploadedCosUrl:', uploadedCosUrl.value || '（空）')
    console.log('最终提交的userAvatar:', submitData.userAvatar || '（空）')
    console.log('userProfile:', submitData.userProfile || '（空）')
    console.log('完整数据：', JSON.stringify(submitData, null, 2))

    // 只有填写了密码才传递密码字段
    if (formData.value.userPassword) {
      submitData.userPassword = formData.value.userPassword
      submitData.checkPassword = formData.value.checkPassword
    }
    const res = await updateMyInfo(submitData)
    if (res.data.code === 0) {
      message.success('修改成功')
      // 清空已上传的COS URL（因为已经保存到数据库）
      uploadedCosUrl.value = ''
      // 重新获取用户信息
      await loginUserStore.fetchLoginUser()
      emit('success')
      visible.value = false
    } else {
      message.error('修改失败：' + res.data.message)
    }
  } catch (error: any) {
    message.error('修改失败：' + (error?.message || '请重试'))
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
  border: 2px solid #d9d9d9;
  transition: all 0.3s;
}

.avatar-upload-mask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #fff;
  opacity: 0;
  transition: opacity 0.3s;
  font-size: 12px;
}

.avatar-upload-mask .anticon {
  font-size: 20px;
  margin-bottom: 4px;
}

.upload-tip {
  color: #1890ff;
  font-size: 12px;
}

.password-tip {
  color: #999;
  font-size: 12px;
  margin-top: -16px;
}
</style>

