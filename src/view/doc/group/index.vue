<template>
  <div class="group-manage">
    <div class="search-form">
      <a-form layout="inline" :model="searchForm">
        <a-form-item label="分组名称">
          <a-input
            v-model:value="searchForm.name"
            placeholder="请输入分组名称"
            allow-clear
            style="width: 200px"
          />
        </a-form-item>
        <a-form-item label="状态">
          <a-select
            v-model:value="searchForm.status"
            placeholder="请选择状态"
            style="width: 120px"
            allow-clear
          >
            <a-select-option :value="1">已发布</a-select-option>
            <a-select-option :value="0">未发布</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item>
          <a-space>
            <a-button type="primary" @click="handleSearch">
              <template #icon><search-outlined /></template>
              查询
            </a-button>
            <a-button @click="handleReset">
              <template #icon><reload-outlined /></template>
              重置
            </a-button>
          </a-space>
        </a-form-item>
      </a-form>
    </div>

    <div class="table-operations">
      <a-button type="primary" @click="showCreateModal">
        <template #icon><plus-outlined /></template>
        新建分组
      </a-button>
    </div>
    <a-table
      :columns="columns"
      :data-source="dataSource"
      :loading="loading"
      :pagination="pagination"
      @change="handleTableChange"
    >
      <template #bodyCell="{ column, record }">
        <template v-if="column.key === 'status'">
          <a-tag :color="record.status === 1 ? 'green' : 'red'">
            {{ record.status === 1 ? '已发布' : '未发布' }}
          </a-tag>
        </template>
        <template v-if="column.key === 'sort'">
          {{ record.sort }}
        </template>
        <template v-if="column.key === 'action'">
          <a-space>
            <a @click="showEditModal(record)">编辑</a>
            <a-divider type="vertical" />
            <a-popconfirm
              title="确定要删除这个分组吗？"
              ok-text="确定"
              cancel-text="取消"
              @confirm="handleDelete(record)"
            >
              <a class="text-danger">删除</a>
            </a-popconfirm>
            <a-divider type="vertical" />
            <a v-if="record.status === 0" @click="handlePublish(record)">发布</a>
            <a v-else @click="handleUnpublish(record)">下架</a>
          </a-space>
        </template>
      </template>
    </a-table>

    <!-- 创建/编辑分组弹窗 -->
    <a-modal
      v-model:visible="modalVisible"
      :title="modalType === 'create' ? '新建分组' : '编辑分组'"
      @ok="handleModalOk"
      @cancel="handleModalCancel"
      okText="确定"
      cancelText="取消"
    >
      <a-form
        ref="formRef"
        :model="formState"
        :rules="rules"
        layout="vertical"
      >
        <a-form-item label="分组名称" name="name">
          <a-input v-model:value="formState.name" placeholder="请输入分组名称" />
        </a-form-item>
        <a-form-item label="分组描述" name="description">
          <a-textarea
            v-model:value="formState.description"
            placeholder="请输入分组描述"
            :rows="4"
          />
        </a-form-item>
        <a-form-item label="排序" name="sort">
          <a-input-number
            v-model:value="formState.sort"
            :min="0"
            placeholder="请输入排序号"
          />
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { message } from 'ant-design-vue'
import { PlusOutlined, SearchOutlined, ReloadOutlined } from '@ant-design/icons-vue'
import type { TablePaginationConfig } from 'ant-design-vue'

interface Group {
  id: string
  name: string
  description: string
  status: number
  sort: number
  createTime: string
  updateTime: string
}

interface FormState {
  id?: string
  name: string
  description: string
  sort: number
}

interface SearchForm {
  name: string
  status: number | undefined
}

const columns = [
  {
    title: '分组名称',
    dataIndex: 'name',
    key: 'name',
  },
  {
    title: '描述',
    dataIndex: 'description',
    key: 'description',
  },
  {
    title: '排序',
    dataIndex: 'sort',
    key: 'sort',
    width: 100,
  },
  {
    title: '状态',
    dataIndex: 'status',
    key: 'status',
  },
  {
    title: '创建时间',
    dataIndex: 'createTime',
    key: 'createTime',
  },
  {
    title: '操作',
    key: 'action',
    width: 200,
  },
]

const dataSource = ref<Group[]>([])
const loading = ref(false)
const modalVisible = ref(false)
const modalType = ref<'create' | 'edit'>('create')
const formRef = ref()
const searchForm = reactive<SearchForm>({
  name: '',
  status: undefined,
})

const formState = reactive<FormState>({
  name: '',
  description: '',
  sort: 0,
})

const rules = {
  name: [{ required: true, message: '请输入分组名称', trigger: 'blur' }],
  description: [{ required: true, message: '请输入分组描述', trigger: 'blur' }],
  sort: [{ required: true, message: '请输入排序号', trigger: 'blur' }],
}

const pagination = reactive({
  current: 1,
  pageSize: 10,
  total: 0,
})

// 获取分组列表
const fetchGroups = async () => {
  loading.value = true
  try {
    // TODO: 调用后端API获取分组列表，传入查询条件
    // 模拟数据
    dataSource.value = [
      {
        id: '1',
        name: '示例分组',
        description: '这是一个示例分组',
        status: 1,
        sort: 1,
        createTime: '2024-01-01 12:00:00',
        updateTime: '2024-01-01 12:00:00',
      },
    ]
    pagination.total = 1
  } catch (error) {
    message.error('获取分组列表失败')
  } finally {
    loading.value = false
  }
}

// 处理查询
const handleSearch = () => {
  pagination.current = 1
  fetchGroups()
}

// 处理重置
const handleReset = () => {
  searchForm.name = ''
  searchForm.status = undefined
  handleSearch()
}

// 表格变化处理
const handleTableChange = (pag: TablePaginationConfig) => {
  pagination.current = pag.current || 1
  pagination.pageSize = pag.pageSize || 10
  fetchGroups()
}

// 显示创建弹窗
const showCreateModal = () => {
  modalType.value = 'create'
  formState.id = undefined
  formState.name = ''
  formState.description = ''
  formState.sort = 0
  modalVisible.value = true
}

// 显示编辑弹窗
const showEditModal = (record: Group) => {
  modalType.value = 'edit'
  formState.id = record.id
  formState.name = record.name
  formState.description = record.description
  formState.sort = record.sort
  modalVisible.value = true
}

// 处理弹窗确认
const handleModalOk = async () => {
  try {
    await formRef.value.validate()
    // TODO: 调用后端API保存分组
    if (modalType.value === 'create') {
      message.success('创建成功')
    } else {
      message.success('更新成功')
    }
    modalVisible.value = false
    fetchGroups()
  } catch (error) {
    console.error('表单验证失败:', error)
  }
}

// 处理弹窗取消
const handleModalCancel = () => {
  modalVisible.value = false
}

// 删除分组
const handleDelete = async (record: Group) => {
  try {
    // TODO: 调用后端API删除分组
    message.success('删除成功')
    fetchGroups()
  } catch (error) {
    message.error('删除失败')
  }
}

// 发布分组
const handlePublish = async (record: Group) => {
  try {
    // TODO: 调用后端API发布分组
    message.success('发布成功')
    fetchGroups()
  } catch (error) {
    message.error('发布失败')
  }
}

// 下架分组
const handleUnpublish = async (record: Group) => {
  try {
    // TODO: 调用后端API下架分组
    message.success('下架成功')
    fetchGroups()
  } catch (error) {
    message.error('下架失败')
  }
}

onMounted(() => {
  fetchGroups()
})
</script>

<style scoped>
.group-manage {
  padding: 24px;
}

.search-form {
  margin-bottom: 16px;
  padding: 24px 24px 24px 0;
  background: #fff;
  border-radius: 2px;
}

.search-form :deep(.ant-form-item) {
  margin-bottom: 0;
}

.table-operations {
  margin-bottom: 16px;
}

.text-danger {
  color: #ff4d4f;
}
</style> 