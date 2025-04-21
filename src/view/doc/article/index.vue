<template>
  <div class="article-manage">
    <a-row :gutter="16">
      <!-- 左侧目录树 -->
      <a-col :span="6">
        <div class="tree-container">
          <div class="tree-header">
            <a-space>
              <a-select
                v-model:value="selectedGroup"
                style="width: 200px"
                placeholder="请选择分组"
                @change="handleGroupChange"
              >
                <a-select-option v-for="group in groups" :key="group.id" :value="group.id">
                  {{ group.name }}
                </a-select-option>
              </a-select>
              <a-dropdown>
                <a-button type="primary">
                  <template #icon><ellipsis-outlined /></template>
                </a-button>
                <template #overlay>
                  <a-menu>
                    <a-menu-item key="create" @click="showCreateModal">新增</a-menu-item>
                    <a-menu-item key="publish" :disabled="checkedKeys.length === 0" @click="handlePublish">发布</a-menu-item>
                    <a-menu-item key="unpublish" :disabled="checkedKeys.length === 0" @click="handleUnpublish">下架</a-menu-item>
                    <a-menu-item :disabled="checkedKeys.length === 0">
                      <a-popconfirm
                        title="确定要删除选中的节点吗？"
                        ok-text="确定"
                        cancel-text="取消"
                        @confirm="handleGroupDelete"
                      >
                        <span>删除</span>
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
              :checkable="true"
              :show-line="true"
              @select="onSelect"
              @check="onCheck"
            >
              <template #title="{ title, type, status }">
                <div class="tree-node-content">
                  <span class="node-status">
                    <a-tag v-if="status === 1" color="success">已发布</a-tag>
                    <a-tag v-else color="error">未发布</a-tag>
                  </span>
                  <span class="node-title">{{ title }}</span>
                  <a-dropdown v-if="type !== 'group'" :trigger="['click']">
                    <a class="ant-dropdown-link" @click.prevent>
                      <more-outlined />
                    </a>
                    <template #overlay>
                      <a-menu>
                        <a-menu-item key="edit" @click="handleEdit">
                          <edit-outlined />
                          编辑
                        </a-menu-item>
                        <a-menu-item key="publish" @click="() => {
                          message.success('发布成功');
                          handlePublish([node.key]);
                        }">
                          <check-circle-outlined />
                          发布
                        </a-menu-item>
                        <a-menu-item key="unpublish" @click="() => {
                          message.success('下架成功');
                          handleUnpublish([node.key]);
                        }">
                          <close-circle-outlined />
                          下架
                        </a-menu-item>
                        <a-popconfirm
                          v-model:visible="deleteConfirmVisible"
                          title="确定要删除这个节点吗？"
                          ok-text="确定"
                          cancel-text="取消"
                          @confirm="() => {
                            message.success('删除成功');
                            handleDelete([node.key]);
                          }"
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
              </template>
            </a-tree>
          </div>
        </div>
      </a-col>

      <!-- 右侧编辑区域 -->
      <a-col :span="18">
        <div class="editor-container" v-if="selectedArticle">
          <div class="editor-header">
            <a-space>
              <h2>{{ selectedArticle.title }}</h2>
              <div class="editor-actions">
                <a-button type="primary" @click="handleSave">保存</a-button>
                <a-button @click="handlePreview">预览</a-button>
              </div>
            </a-space>
          </div>
          <div class="editor-content">
            <MdEditor v-model="selectedArticle.content" />
          </div>
        </div>
        <div class="empty-container" v-else>
          <a-empty description="请选择或创建一篇文章" />
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
        <a-form-item label="标题" name="title">
          <a-input v-model:value="formState.title" placeholder="请输入标题" />
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
import { ref, reactive, onMounted, watch } from 'vue'
import { message } from 'ant-design-vue'
import { PlusOutlined, EllipsisOutlined, CheckCircleOutlined, CloseCircleOutlined, FolderOutlined, FolderOpenOutlined, EditOutlined, DeleteOutlined, MoreOutlined } from '@ant-design/icons-vue'
import { MdEditor } from 'md-editor-v3'
import 'md-editor-v3/lib/style.css'

interface Group {
  id: string
  name: string
}

interface TreeNode {
  key: string
  title: string
  type: 'directory' | 'article'
  children?: TreeNode[]
  parentKey?: string
  sort: number
  status: number
}

interface Article {
  id: string
  title: string
  content: string
  type: 'directory' | 'article'
  parentId?: string
  sort: number
}

interface FormState {
  type: 'directory' | 'article'
  title: string
  sort: number
  parentId?: string
}

// 状态定义
const groups = ref<Group[]>([])
const selectedGroup = ref<string>('')
const treeData = ref<TreeNode[]>([])
const selectedKeys = ref<string[]>([])
const checkedKeys = ref<string[]>([])
const selectedArticle = ref<Article | null>(null)
const modalVisible = ref(false)
const modalType = ref<'create' | 'edit'>('create')
const formRef = ref()
const deleteConfirmVisible = ref(false)

const formState = reactive<FormState>({
  type: 'article',
  title: '',
  sort: 0,
})

const rules = {
  type: [{ required: true, message: '请选择类型' }],
  title: [{ required: true, message: '请输入标题' }],
  sort: [{ required: true, message: '请输入排序号' }],
  parentId: [],
}

// 获取分组列表
const fetchGroups = async () => {
  try {
    // TODO: 调用后端API获取分组列表
    groups.value = [
      { id: '1', name: '示例分组' },
    ]
    // 自动选择第一个分组
    if (groups.value.length > 0 && !selectedGroup.value) {
      selectedGroup.value = groups.value[0].id
    }
  } catch (error) {
    message.error('获取分组列表失败')
  }
}

// 获取文档树
const fetchDocTree = async () => {
  if (!selectedGroup.value) return
  try {
    // TODO: 调用后端API获取文档树
    treeData.value = [
      {
        key: '1',
        title: '示例目录',
        type: 'directory',
        sort: 1,
        status: 1,
        children: [
          {
            key: '2',
            title: '示例文章',
            type: 'article',
            sort: 1,
            status: 1,
          },
        ],
      },
    ]
  } catch (error) {
    message.error('获取文档树失败')
  }
}

// 监听分组选择变化
watch(selectedGroup, (newValue) => {
  if (newValue) {
    fetchDocTree()
  }
})

// 处理分组变更
const handleGroupChange = (value: string) => {
  selectedGroup.value = value
  selectedKeys.value = []
  checkedKeys.value = []
  selectedArticle.value = null
}

// 选择节点
const onSelect = async (selectedKeys: string[], info: any) => {
  if (selectedKeys.length === 0) return
  const node = info.node
  if (node.type === 'article') {
    try {
      // TODO: 调用后端API获取文章详情
      selectedArticle.value = {
        id: node.key,
        title: node.title,
        content: '# 示例文章\n这是一篇示例文章的内容',
        type: 'article',
        sort: node.sort,
        parentId: node.parentKey,
      }
    } catch (error) {
      message.error('获取文章详情失败')
    }
  } else {
    selectedArticle.value = null
  }
}

// 处理勾选
const onCheck = (checkedKeys: string[], info: any) => {
  console.log('checkedKeys:', checkedKeys)
  console.log('info:', info)
}

// 显示创建弹窗
const showCreateModal = () => {
  if (!selectedGroup.value) {
    message.warning('请先选择分组')
    return
  }
  modalType.value = 'create'
  formState.type = 'article'
  formState.title = ''
  formState.sort = 0
  formState.parentId = selectedKeys.value[0]
  modalVisible.value = true
}

// 处理编辑
const handleEdit = (node: { key: string; title: string; type: 'directory' | 'article'; parentKey?: string; sort: number }) => {
  modalType.value = 'edit'
  formState.title = node.title
  formState.sort = node.sort
  formState.parentId = node.parentKey
  formState.type = node.type
  modalVisible.value = true
  // 如果是文章节点，同时加载文章内容
  if (node.type === 'article') {
    try {
      // TODO: 调用后端API获取文章详情
      selectedArticle.value = {
        id: node.key,
        title: node.title,
        content: '# 示例文章\n这是一篇示例文章的内容',
        type: 'article',
        sort: node.sort,
        parentId: node.parentKey,
      }
    } catch (error) {
      message.error('获取文章详情失败')
    }
  } else {
    selectedArticle.value = null
  }
}

// 处理删除
const handleDelete = (keys: string[]) => {
  if (!selectedGroup.value) {
    message.warning('请先选择分组')
    return
  }
  // TODO: 实现删除功能
  console.log('Deleting node with key:', keys)
}

// 处理分组删除
const handleGroupDelete = async () => {
  if (!selectedGroup.value) {
    message.warning('请先选择分组')
    return
  }
  try {
    // TODO: 调用后端API删除分组
    message.success('删除成功')
    fetchDocTree()
  } catch (error) {
    message.error('删除失败')
  }
}

// 处理保存文章
const handleSave = async () => {
  if (!selectedArticle.value) return
  try {
    // TODO: 调用后端API保存文章
    message.success('保存成功')
  } catch (error) {
    message.error('保存失败')
  }
}

// 处理预览
const handlePreview = () => {
  // TODO: 实现预览功能
}

// 处理弹窗确认
const handleModalOk = async () => {
  try {
    await formRef.value.validate()
    // TODO: 调用后端API保存节点
    if (modalType.value === 'create') {
      message.success('创建成功')
    } else {
      message.success('更新成功')
    }
    modalVisible.value = false
    fetchDocTree()
  } catch (error) {
    console.error('表单验证失败:', error)
  }
}

// 处理弹窗取消
const handleModalCancel = () => {
  modalVisible.value = false
}

// 处理发布
const handlePublish = (keys: string[]) => {
  if (!selectedGroup.value) {
    message.warning('请先选择分组')
    return
  }
  // TODO: 实现发布功能
  message.success('发布成功')
}

// 处理下架
const handleUnpublish = (keys: string[]) => {
  if (!selectedGroup.value) {
    message.warning('请先选择分组')
    return
  }
  // TODO: 实现下架功能
  message.success('下架成功')
}

onMounted(() => {
  fetchGroups()
})
</script>

<style scoped>
.article-manage {
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

.editor-container {
  background: #fff;
  padding: 16px 0 16px 16px;
  border-radius: 2px;
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.editor-header {
  margin-bottom: 16px;
  flex-shrink: 0;
}

.editor-header .ant-space {
  width: 100%;
  justify-content: space-between;
}

.editor-actions {
  display: flex;
  gap: 8px;
}

.editor-content {
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

:deep(.ant-tree) {
  background: transparent;
}

:deep(.ant-tree-node-selected) {
  background-color: #e6f7ff !important;
}

:deep(.ant-tree-node-content-wrapper) {
  flex: 1;
}

:deep(.ant-tree-node-content-wrapper:hover) {
  background-color: #f5f5f5 !important;
}

:deep(.ant-select) {
  width: 200px;
  flex-shrink: 0;
}

:deep(.ant-tree-checkbox) {
  margin-right: 8px;
}

:deep(.ant-tree-node-content-wrapper) {
  padding: 0 4px;
}

.tree-node-content {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
}

.node-status {
  display: flex;
  align-items: center;
  min-width: 60px;
}

.node-status :deep(.ant-tag) {
  margin: 0;
  padding: 0 8px;
  height: 22px;
  line-height: 20px;
  font-size: 12px;
}

.node-title {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
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

:deep(.ant-space) {
  flex-wrap: nowrap;
  gap: 0 !important;
}

:deep(.ant-space-item) {
  margin-right: 8px;
}
</style> 