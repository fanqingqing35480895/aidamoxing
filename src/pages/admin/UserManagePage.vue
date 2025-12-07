<template>
  <div id="userManagePage">
    <!-- 搜索表单 -->
    <a-form layout="inline" :model="searchParams" @finish="doSearch">
      <a-form-item label="账号">
        <a-input v-model:value="searchParams.userAccount" placeholder="输入账号" />
      </a-form-item>
      <a-form-item label="用户名">
        <a-input v-model:value="searchParams.userName" placeholder="输入用户名" />
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
    >
      <template #bodyCell="{ column, record }">
        <template v-if="column.dataIndex === 'userAvatar'">
          <a-image :src="record.userAvatar" :width="120" />
        </template>
        <template v-else-if="column.dataIndex === 'userRole'">
          <div v-if="record.userRole === 'admin'">
            <a-tag color="green">管理员</a-tag>
          </div>
          <div v-else>
            <a-tag color="blue">普通用户</a-tag>
          </div>
        </template>
        <template v-else-if="column.dataIndex === 'createTime'">
          {{ dayjs(record.createTime).format('YYYY-MM-DD HH:mm:ss') }}
        </template>
        <template v-else-if="column.key === 'action'">
          <a-button danger @click="doDelete(record.id)">删除</a-button>
        </template>
      </template>
    </a-table>
  </div>
</template>
<script lang="ts" setup>
import { computed, onMounted, reactive, ref } from 'vue'
import { deleteUser, listUserVoByPage } from '@/api/userController.ts'
import { message } from 'ant-design-vue'
import dayjs from 'dayjs'

const columns = [
  {
    title: 'id',
    dataIndex: 'id',
  },
  {
    title: '账号',
    dataIndex: 'userAccount',
  },
  {
    title: '用户名',
    dataIndex: 'userName',
  },
  {
    title: '头像',
    dataIndex: 'userAvatar',
  },
  {
    title: '简介',
    dataIndex: 'userProfile',
  },
  {
    title: '用户角色',
    dataIndex: 'userRole',
  },
  {
    title: '创建时间',
    dataIndex: 'createTime',
  },
  {
    title: '操作',
    key: 'action',
  },
]

// 展示的数据
const data = ref<API.UserVO[]>([])
const total = ref(0)

// 搜索条件
const searchParams = reactive<API.UserQueryRequest>({
  pageNum: 1,
  pageSize: 10,
})

// 获取数据
const fetchData = async () => {
  const res = await listUserVoByPage({
    ...searchParams,
  })
  if (res.data.data) {
    data.value = res.data.data.records ?? []
    total.value = res.data.data.totalRow ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
}

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

// 表格分页变化时的操作
const doTableChange = (page: { current: number; pageSize: number }) => {
  searchParams.pageNum = page.current
  searchParams.pageSize = page.pageSize
  fetchData()
}

// 搜索数据
const doSearch = () => {
  // 重置页码
  searchParams.pageNum = 1
  fetchData()
}

// 删除数据
const doDelete = async (id: string) => {
  if (!id) {
    return
  }
  const res = await deleteUser({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
    // 刷新数据
    fetchData()
  } else {
    message.error('删除失败')
  }
}

// 页面加载时请求一次
onMounted(() => {
  fetchData()
})
</script>

<style scoped>
#userManagePage {
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

:deep(.ant-input) {
  background: rgba(0, 0, 0, 0.6) !important;
  border-color: rgba(0, 240, 255, 0.3) !important;
  color: #ffffff !important;
}

:deep(.ant-input:focus),
:deep(.ant-input-focused) {
  border-color: #00f0ff !important;
  box-shadow: 0 0 10px rgba(0, 240, 255, 0.3) !important;
}

:deep(.ant-input::placeholder) {
  color: rgba(0, 212, 255, 0.5) !important;
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
}

:deep(.ant-table-tbody > tr:hover > td) {
  background: rgba(0, 240, 255, 0.15) !important;
}

:deep(.ant-table-tbody > tr.ant-table-row-selected > td),
:deep(.ant-table-tbody > tr.ant-table-row-selected:hover > td) {
  background: rgba(0, 240, 255, 0.2) !important;
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

:deep(.ant-divider) {
  border-color: rgba(0, 240, 255, 0.3);
}

:deep(.ant-tag) {
  background: rgba(0, 240, 255, 0.1);
  border-color: rgba(0, 240, 255, 0.3);
  color: #00f0ff;
}

:deep(.ant-image) {
  border: 1px solid rgba(0, 240, 255, 0.3);
  border-radius: 4px;
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

/* 固定列在hover和选中状态下的背景色 */
:deep(.ant-table-tbody > tr:hover .ant-table-cell-fix-left),
:deep(.ant-table-tbody > tr:hover .ant-table-cell-fix-right) {
  background: rgba(0, 240, 255, 0.15) !important;
}

:deep(.ant-table-tbody > tr.ant-table-row-selected .ant-table-cell-fix-left),
:deep(.ant-table-tbody > tr.ant-table-row-selected .ant-table-cell-fix-right),
:deep(.ant-table-tbody > tr.ant-table-row-selected:hover .ant-table-cell-fix-left),
:deep(.ant-table-tbody > tr.ant-table-row-selected:hover .ant-table-cell-fix-right) {
  background: rgba(0, 240, 255, 0.2) !important;
}
</style>
