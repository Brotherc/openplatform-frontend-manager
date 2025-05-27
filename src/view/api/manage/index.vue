<template>
  <div class="api-manage">
    <a-row :gutter="16">
      <!-- 左侧模块树 -->
      <a-col :span="6">
        <div class="tree-container">
          <div class="tree-header">
            <a-space>
              <a-button type="primary" @click="showCreateModal">
                <template #icon><plus-outlined /></template>
                新增
              </a-button>
              <a-dropdown>
                <a-button type="primary" :disabled="checkedKeys.length === 0">
                  <template #icon><ellipsis-outlined /></template>
                </a-button>
                <template #overlay>
                  <a-menu>
                    <a-menu-item key="publish" :disabled="checkedKeys.length === 0" @click="() => handlePublish(checkedKeys)">
                      <check-circle-outlined />
                      发布
                    </a-menu-item>
                    <a-menu-item key="unpublish" :disabled="checkedKeys.length === 0" @click="() => handleUnpublish(checkedKeys)">
                      <close-circle-outlined />
                      下架
                    </a-menu-item>
                    <a-menu-item :disabled="checkedKeys.length === 0">
                      <a-popconfirm
                        title="确定要删除选中的节点吗？"
                        ok-text="确定"
                        cancel-text="取消"
                        @confirm="() => handleBatchDelete(checkedKeys)"
                      >
                        <span>
                          <delete-outlined />
                          删除
                        </span>
                      </a-popconfirm>
                    </a-menu-item>
                  </a-menu>
                </template>
              </a-dropdown>
            </a-space>
          </div>
          <div class="tree-content">
            <a-tree
              v-model:selectedKeys="selectedKeys"
              v-model:checkedKeys="checkedKeys"
              :tree-data="treeData"
              :block-node="true"
              :checkable="true"
              @select="onSelect"
              @check="onCheck"
            >
              <template #title="{ title, type, dataRef }">
                <div class="tree-node-content">
                  <folder-outlined v-if="type === 1" style="margin-right: 8px" />
                  <api-outlined v-else style="margin-right: 8px" />
                  <span class="node-title" :title="title">{{ title }}</span>
                  <div class="node-actions" @click.stop>
                    <a-dropdown :trigger="['click']">
                      <a class="ant-dropdown-link" @click.prevent.stop>
                        <more-outlined />
                      </a>
                      <template #overlay>
                        <a-menu>
                          <a-menu-item key="edit" @click="() => handleEdit(dataRef)">
                            <edit-outlined />
                            编辑
                          </a-menu-item>
                          <a-menu-item key="publish" @click="() => handlePublish([dataRef.key])">
                            <check-circle-outlined />
                            发布
                          </a-menu-item>
                          <a-menu-item key="unpublish" @click="() => handleUnpublish([dataRef.key])">
                            <close-circle-outlined />
                            下架
                          </a-menu-item>
                          <a-popconfirm
                            title="确定要删除这个节点吗？"
                            ok-text="确定"
                            cancel-text="取消"
                            @confirm="() => handleDelete(dataRef.key)"
                          >
                            <a-menu-item key="delete">
                              <delete-outlined />
                              删除
                            </a-menu-item>
                          </a-popconfirm>
                        </a-menu>
                      </template>
                    </a-dropdown>
                  </div>
                </div>
              </template>
            </a-tree>
          </div>
        </div>
      </a-col>

      <!-- 右侧API详情 -->
      <a-col :span="18">
        <div class="api-detail" v-if="selectedApi">
          <div class="detail-header">
            <div class="detail-actions">
              <a-button type="primary" @click="handleSaveApi">保存</a-button>
            </div>
          </div>
          <div class="detail-content">
            <a-form
              :model="apiForm"
              layout="vertical"
            >
              <a-row :gutter="16">
                <a-col :span="12">
                  <a-form-item label="API名称">
                    <a-input v-model:value="apiForm.name" placeholder="请输入API名称" />
                  </a-form-item>
                </a-col>
                <a-col :span="12">
                  <a-form-item label="中文名称">
                    <a-input v-model:value="apiForm.nameCn" placeholder="请输入中文名称" />
                  </a-form-item>
                </a-col>
              </a-row>
              <a-row :gutter="16">
                <a-col :span="24">
                  <a-form-item label="描述">
                    <a-textarea v-model:value="apiForm.description" placeholder="请输入API描述" :rows="4" />
                  </a-form-item>
                </a-col>
              </a-row>
              <a-row :gutter="16">
                <a-col :span="12">
                  <a-form-item label="请求方法">
                    <a-select v-model:value="apiForm.method" placeholder="请选择请求方法">
                      <a-select-option value="GET">GET</a-select-option>
                      <a-select-option value="POST">POST</a-select-option>
                      <a-select-option value="PUT">PUT</a-select-option>
                      <a-select-option value="DELETE">DELETE</a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :span="12">
                  <a-form-item label="请求路径">
                    <a-input v-model:value="apiForm.path" placeholder="请输入请求路径" />
                  </a-form-item>
                </a-col>
              </a-row>
            </a-form>
          </div>
        </div>
        <div class="empty-container" v-else>
          <a-empty description="请选择一个API" />
        </div>
      </a-col>
    </a-row>

    <!-- 创建/编辑弹窗 -->
    <a-modal
      v-model:visible="modalVisible"
      :title="modalType === 'create' ? '新建' : '编辑'"
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
        <a-form-item label="类型" name="type">
          <a-radio-group v-model:value="formState.type" :disabled="modalType === 'edit'">
            <a-radio :value="1">模块</a-radio>
            <a-radio :value="2">API</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item label="父级节点" name="parentId">
          <a-tree-select
            v-model:value="formState.parentId"
            style="width: 100%"
            :dropdown-style="{ maxHeight: '400px', overflow: 'auto' }"
            :tree-data="treeData"
            placeholder="请选择父级节点"
            :field-names="{ label: 'title', value: 'key', children: 'children' }"
            :tree-default-expand-all="true"
            allowClear
          />
        </a-form-item>
        <a-form-item label="排序" name="sort">
          <a-input-number
            v-model:value="formState.sort"
            :min="0"
            placeholder="请输入排序号"
          />
        </a-form-item>
        <template v-if="formState.type === 2">
          <a-form-item label="名称" name="name">
            <a-input v-model:value="formState.name" placeholder="请输入名称" />
          </a-form-item>
          <a-form-item label="中文名称" name="nameCn">
            <a-input v-model:value="formState.nameCn" placeholder="请输入中文名称" />
          </a-form-item>
          <a-form-item label="描述" name="description">
            <a-textarea v-model:value="formState.description" placeholder="请输入描述" />
          </a-form-item>
        </template>
      </a-form>
    </a-modal>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { message } from 'ant-design-vue'
import {
  PlusOutlined,
  FolderOutlined,
  ApiOutlined,
  EditOutlined,
  DeleteOutlined,
  MoreOutlined,
  EllipsisOutlined,
  CheckCircleOutlined,
  CloseCircleOutlined
} from '@ant-design/icons-vue'
import axios from 'axios'

interface TreeNode {
  key: string
  title: string
  type: 1 | 2  // 1-模块，2-API
  children?: TreeNode[]
  parentKey?: string
  sort: number
  name?: string
  nameCn?: string
  description?: string
}

interface FormState {
  type: 1 | 2
  title: string
  sort: number
  parentId?: string
  name?: string
  nameCn?: string
  description?: string
  id?: string
}

interface ApiForm {
  name: string
  nameCn: string
  method: string
  path: string
  description: string
}

// 状态定义
const treeData = ref<TreeNode[]>([])
const selectedKeys = ref<string[]>([])
const checkedKeys = ref<string[]>([])
const selectedApi = ref<TreeNode | null>(null)
const modalVisible = ref(false)
const modalType = ref<'create' | 'edit'>('create')
const formRef = ref()

const formState = reactive<FormState>({
  type: 1,
  title: '',
  sort: 0
})

const apiForm = reactive<ApiForm>({
  name: '',
  nameCn: '',
  method: 'GET',
  path: '',
  description: ''
})

const rules = {
  type: [{ required: true, message: '请选择类型' }],
  title: [{ required: true, message: '请输入标题' }],
  sort: [{ required: true, message: '请输入排序号' }],
  name: [{ required: true, message: '请输入名称' }],
  nameCn: [{ required: true, message: '请输入中文名称' }]
}

// 获取模块树
const fetchTree = async () => {
  try {
    // TODO: 调用后端API获取树数据
    // 模拟数据
    treeData.value = [
      {
        key: '1',
        title: '用户模块',
        type: 1,
        sort: 1,
        children: [
          {
            key: '1-1',
            title: '获取用户信息',
            type: 2,
            sort: 1,
            name: 'getUserInfo',
            nameCn: '获取用户信息',
            description: '获取当前登录用户的信息'
          }
        ]
      }
    ]
  } catch (error) {
    console.error('获取模块树失败:', error)
    message.error('获取模块树失败，请稍后重试')
  }
}

// 选择节点
const onSelect = (selectedKeys: string[], info: any) => {
  const node = info.node
  if (node.dataRef.type === 2) {
    selectedApi.value = node.dataRef
    apiForm.name = node.dataRef.name || ''
    apiForm.nameCn = node.dataRef.nameCn || ''
    apiForm.method = node.dataRef.method || 'GET'
    apiForm.path = node.dataRef.path || ''
    apiForm.description = node.dataRef.description || ''
  } else {
    selectedApi.value = null
  }
}

// 显示创建弹窗
const showCreateModal = () => {
  modalType.value = 'create'
  formState.type = 1
  formState.title = ''
  formState.sort = 0
  formState.parentId = undefined
  formState.name = ''
  formState.nameCn = ''
  formState.description = ''
  modalVisible.value = true
}

// 处理编辑
const handleEdit = (node: any) => {
  modalType.value = 'edit'
  formState.id = node.key
  formState.type = node.type
  formState.title = node.title
  formState.sort = node.sort
  formState.parentId = node.parentKey
  formState.name = node.name
  formState.nameCn = node.nameCn
  formState.description = node.description
  modalVisible.value = true
}

// 处理删除
const handleDelete = async (key: string) => {
  try {
    // TODO: 调用后端API删除
    message.success('删除成功')
    fetchTree()
  } catch (error) {
    console.error('删除失败:', error)
    message.error('删除失败，请稍后重试')
  }
}

// 处理弹窗确认
const handleModalOk = async () => {
  try {
    await formRef.value.validate()
  } catch (error) {
    return
  }

  try {
    if (modalType.value === 'create') {
      // TODO: 调用后端API创建
      message.success('创建成功')
    } else {
      // TODO: 调用后端API更新
      message.success('更新成功')
    }
    modalVisible.value = false
    fetchTree()
  } catch (error) {
    console.error('操作失败:', error)
    message.error('操作失败，请稍后重试')
  }
}

// 处理弹窗取消
const handleModalCancel = () => {
  modalVisible.value = false
}

// 保存API
const handleSaveApi = async () => {
  if (!selectedApi.value) return
  try {
    // TODO: 调用后端API保存
    message.success('保存成功')
    fetchTree()
  } catch (error) {
    console.error('保存失败:', error)
    message.error('保存失败，请稍后重试')
  }
}

// 处理勾选
const onCheck = (checked: string[], info: any) => {
  console.log('checked:', checked)
  console.log('info:', info)
}

// 处理批量删除
const handleBatchDelete = async (keys: string[]) => {
  try {
    // TODO: 调用后端API批量删除
    message.success('删除成功')
    fetchTree()
  } catch (error) {
    console.error('删除失败:', error)
    message.error('删除失败，请稍后重试')
  }
}

// 处理发布
const handlePublish = async (keys: string[]) => {
  try {
    // TODO: 调用后端API发布
    message.success('发布成功')
    fetchTree()
  } catch (error) {
    console.error('发布失败:', error)
    message.error('发布失败，请稍后重试')
  }
}

// 处理下架
const handleUnpublish = async (keys: string[]) => {
  try {
    // TODO: 调用后端API下架
    message.success('下架成功')
    fetchTree()
  } catch (error) {
    console.error('下架失败:', error)
    message.error('下架失败，请稍后重试')
  }
}

onMounted(() => {
  fetchTree()
})
</script>

<style scoped>
.api-manage {
  height: 100%;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.tree-container {
  background: #fff;
  padding: 16px 0 16px 8px;
  border-radius: 2px;
  height: 100%;
  display: flex;
  flex-direction: column;
  position: relative;
  z-index: 1;
  overflow: hidden;
  min-width: 300px;
  border-right: 1px solid #f0f0f0;
}

.tree-header {
  margin-bottom: 16px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
}

.tree-content {
  flex: 1;
  overflow: auto;
  min-width: 0;
}

.api-detail {
  background: #fff;
  padding: 16px 0 16px 16px;
  border-radius: 2px;
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.detail-header {
  margin-bottom: 16px;
  flex-shrink: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.detail-content {
  flex: 1;
  overflow: auto;
  min-width: 0;
}

.empty-container {
  background: #fff;
  padding: 16px;
  border-radius: 2px;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.tree-node-content {
  display: flex;
  align-items: center;
  width: 100%;
}

.node-title {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.node-actions {
  padding-left: 8px;
}

:deep(.ant-dropdown-link) {
  color: rgba(0, 0, 0, 0.45);
  padding: 4px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

:deep(.ant-dropdown-link:hover) {
  color: #1890ff;
}

:deep(.ant-row) {
  margin: 0 !important;
  height: 100%;
  flex-wrap: nowrap;
}

:deep(.ant-col) {
  height: 100%;
  display: flex;
  flex-direction: column;
}

:deep(.ant-col-6) {
  flex: 0 0 300px;
  max-width: 300px;
  min-width: 300px;
}

:deep(.ant-col-18) {
  flex: 1;
  min-width: 400px;
}
</style> 