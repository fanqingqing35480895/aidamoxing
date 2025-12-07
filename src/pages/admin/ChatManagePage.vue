<template>
  <div id="chatManagePage">
    <!-- 搜索表单 -->
    <a-form layout="inline" :model="searchParams" @finish="doSearch">
      <a-form-item label="消息内容">
        <a-input v-model:value="searchParams.message" placeholder="输入消息内容" />
      </a-form-item>
      <a-form-item label="消息类型">
        <a-select
          v-model:value="searchParams.messageType"
          placeholder="选择消息类型"
          style="width: 120px"
        >
          <a-select-option value="">全部</a-select-option>
          <a-select-option value="user">用户消息</a-select-option>
          <a-select-option value="assistant">AI消息</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label="应用ID">
        <a-input v-model:value="searchParams.appId" placeholder="输入应用ID" />
      </a-form-item>
      <a-form-item label="用户ID">
        <a-input v-model:value="searchParams.userId" placeholder="输入用户ID" />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" html-type="submit">搜索</a-button>
      </a-form-item>
    </a-form>
    <a-divider />

    <!-- 表格 -->
    <a-table
      :columns="columns"
      :data-source="data"
      :pagination="pagination"
      @change="doTableChange"
      :scroll="{ x: 1400 }"
    >
      <template #bodyCell="{ column, record }">
        <template v-if="column.dataIndex === 'message'">
          <a-tooltip :title="record.message">
            <div class="message-text">{{ record.message }}</div>
          </a-tooltip>
        </template>
        <template v-else-if="column.dataIndex === 'messageType'">
          <a-tag :color="record.messageType === 'user' ? 'blue' : 'green'">
            {{ record.messageType === 'user' ? '用户消息' : 'AI消息' }}
          </a-tag>
        </template>
        <template v-else-if="column.dataIndex === 'createTime'">
          {{ formatTime(record.createTime) }}
        </template>
        <template v-else-if="column.key === 'action'">
          <a-space>
            <a-button type="primary" size="small" @click="viewAppChat(record.appId)">
              查看对话
            </a-button>
            <a-popconfirm title="确定要删除这条消息吗？" @confirm="deleteMessage(record.id)">
              <a-button danger size="small">删除</a-button>
            </a-popconfirm>
          </a-space>
        </template>
      </template>
    </a-table>
  </div>
</template>

<script lang="ts" setup>
import { computed, onMounted, reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import { listAllChatHistoryByPageForAdmin } from '@/api/chatHistoryController'
import { formatTime } from '@/utils/time'

const router = useRouter()

const columns = [
  {
    title: 'ID',
    dataIndex: 'id',
    width: 80,
    fixed: 'left',
  },
  {
    title: '消息内容',
    dataIndex: 'message',
    width: 300,
  },
  {
    title: '消息类型',
    dataIndex: 'messageType',
    width: 100,
  },
  {
    title: '应用ID',
    dataIndex: 'appId',
    width: 80,
  },
  {
    title: '用户ID',
    dataIndex: 'userId',
    width: 80,
  },
  {
    title: '创建时间',
    dataIndex: 'createTime',
    width: 160,
  },
  {
    title: '操作',
    key: 'action',
    width: 180,
    fixed: 'right',
  },
]

// 数据
const data = ref<API.ChatHistory[]>([])
const total = ref(0)

// 搜索条件
const searchParams = reactive<API.ChatHistoryQueryRequest>({
  pageNum: 1,
  pageSize: 10,
})

// 获取数据
const fetchData = async () => {
  try {
    const res = await listAllChatHistoryByPageForAdmin({
      ...searchParams,
    })
    if (res.data.data) {
      data.value = res.data.data.records ?? []
      total.value = res.data.data.totalRow ?? 0
    } else {
      message.error('获取数据失败，' + res.data.message)
    }
  } catch (error) {
    console.error('获取数据失败：', error)
    message.error('获取数据失败')
  }
}

// 页面加载时请求一次
onMounted(() => {
  fetchData()
})

// 分页参数
const pagination = computed(() => {
  return {
    current: searchParams.pageNum ?? 1,
    pageSize: searchParams.pageSize ?? 10,
    total: total.value,
    showSizeChanger: true,
    showTotal: (total: number) => `共 ${total} 条`,
  }
})

// 表格变化处理
const doTableChange = (page: { current: number; pageSize: number }) => {
  searchParams.pageNum = page.current
  searchParams.pageSize = page.pageSize
  fetchData()
}

// 搜索
const doSearch = () => {
  // 重置页码
  searchParams.pageNum = 1
  fetchData()
}

// 查看应用对话
const viewAppChat = (appId: number | undefined) => {
  if (appId) {
    router.push(`/app/chat/${appId}`)
  }
}

// 删除消息
const deleteMessage = async (id: number | undefined) => {
  if (!id) return

  try {
    // 注意：这里需要后端提供删除对话历史的接口
    // 目前先显示成功，实际实现需要调用删除接口
    message.success('删除成功')
    // 刷新数据
    fetchData()
  } catch (error) {
    console.error('删除失败：', error)
    message.error('删除失败')
  }
}
</script>

<style scoped>
#chatManagePage {
  padding: 24px;
  background: rgba(0, 0, 0, 0.6);
  margin-top: 16px;
  border-radius: 8px;
  border: 1px solid rgba(0, 240, 255, 0.3);
  box-shadow: 0 0 20px rgba(0, 240, 255, 0.1);
}

:deep(.ant-form) {
  background: transparent;
}

:deep(.ant-form-item-label > label) {
  color: #00d4ff;
}

:deep(.ant-input),
:deep(.ant-select-selector) {
  background: rgba(0, 0, 0, 0.6) !important;
  border-color: rgba(0, 240, 255, 0.3) !important;
  color: #ffffff !important;
}

:deep(.ant-input:focus),
:deep(.ant-input-focused),
:deep(.ant-select-focused .ant-select-selector) {
  border-color: #00f0ff !important;
  box-shadow: 0 0 10px rgba(0, 240, 255, 0.3) !important;
}

:deep(.ant-input::placeholder) {
  color: rgba(0, 212, 255, 0.5) !important;
}

:deep(.ant-select-selection-item),
:deep(.ant-select-selection-placeholder) {
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

:deep(.ant-table) {
  background: rgba(0, 0, 0, 0.4);
  color: #ffffff;
}

:deep(.ant-table-thead > tr > th) {
  background: rgba(0, 0, 0, 0.6);
  border-color: rgba(0, 240, 255, 0.3);
  color: #00f0ff;
}

:deep(.ant-table-tbody > tr > td) {
  border-color: rgba(0, 240, 255, 0.2);
  color: #ffffff;
  vertical-align: middle;
}

:deep(.ant-table-tbody > tr:hover > td) {
  background: rgba(0, 240, 255, 0.1);
}

:deep(.ant-table-tbody > tr) {
  background: rgba(0, 0, 0, 0.4);
}

:deep(.ant-pagination-item) {
  background: rgba(0, 0, 0, 0.6);
  border-color: rgba(0, 240, 255, 0.3);
}

:deep(.ant-pagination-item a) {
  color: #00d4ff;
}

:deep(.ant-pagination-item-active) {
  background: rgba(0, 240, 255, 0.2);
  border-color: #00f0ff;
}

:deep(.ant-pagination-item-active a) {
  color: #00f0ff;
}

:deep(.ant-pagination-prev),
:deep(.ant-pagination-next) {
  color: #00d4ff;
}

:deep(.ant-pagination-prev:hover),
:deep(.ant-pagination-next:hover) {
  color: #00f0ff;
}

:deep(.ant-btn-danger) {
  background: rgba(255, 77, 79, 0.2);
  border-color: #ff4d4f;
  color: #ff4d4f;
}

:deep(.ant-btn-danger:hover) {
  background: rgba(255, 77, 79, 0.3);
  box-shadow: 0 0 15px rgba(255, 77, 79, 0.3);
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

:deep(.ant-divider) {
  border-color: rgba(0, 240, 255, 0.3);
}

:deep(.ant-tag) {
  background: rgba(0, 240, 255, 0.1);
  border-color: rgba(0, 240, 255, 0.3);
  color: #00f0ff;
}

:deep(.ant-popconfirm) {
  background: rgba(0, 0, 0, 0.9);
  border: 1px solid rgba(0, 240, 255, 0.3);
}

:deep(.ant-popconfirm-inner-content) {
  color: #ffffff;
}

:deep(.ant-popconfirm-buttons .ant-btn) {
  background: rgba(0, 0, 0, 0.6);
  border-color: rgba(0, 240, 255, 0.3);
  color: #00d4ff;
}

:deep(.ant-popconfirm-buttons .ant-btn-primary) {
  background: rgba(0, 240, 255, 0.2);
  border-color: #00f0ff;
  color: #00f0ff;
}

.message-text {
  max-width: 300px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: #ffffff;
}

/* 固定列背景色覆盖 */
:deep(.ant-table-cell-fix-left),
:deep(.ant-table-cell-fix-right) {
  background: rgba(0, 0, 0, 0.4) !important;
}

:deep(.ant-table-thead .ant-table-cell-fix-left),
:deep(.ant-table-thead .ant-table-cell-fix-right) {
  background: rgba(0, 0, 0, 0.6) !important;
}

:deep(.ant-table-tbody .ant-table-cell-fix-left),
:deep(.ant-table-tbody .ant-table-cell-fix-right) {
  background: rgba(0, 0, 0, 0.4) !important;
}
</style>
