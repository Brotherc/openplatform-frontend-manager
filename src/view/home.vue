<template>
  <a-layout style="min-height: 100vh">
    <a-layout-sider v-model:collapsed="collapsed" :trigger="null" collapsible>
      <div class="logo" />
      <a-menu
        v-model:selectedKeys="selectedKeys"
        v-model:openKeys="openKeys"
        theme="dark"
        mode="inline"
        @click="handleMenuClick"
      >
        <a-sub-menu key="doc-center">
          <template #title>
            <span>
              <book-outlined />
              <span>文档中心</span>
            </span>
          </template>
          <a-menu-item key="group">
            <folder-outlined />
            <span>分组管理</span>
          </a-menu-item>
          <a-menu-item key="catalog">
            <folder-open-outlined />
            <span>目录管理</span>
          </a-menu-item>
          <a-menu-item key="article">
            <file-text-outlined />
            <span>文章管理</span>
          </a-menu-item>
        </a-sub-menu>
      </a-menu>
    </a-layout-sider>
    <a-layout>
      <a-layout-header style="background: #fff; padding: 0">
        <menu-unfold-outlined
            v-if="collapsed"
            class="trigger"
            @click="() => (collapsed = !collapsed)"
        />
        <menu-fold-outlined v-else class="trigger" @click="() => (collapsed = !collapsed)" />
      </a-layout-header>
      <a-layout-content
          :style="{ margin: '24px 16px', padding: '24px', background: '#fff', minHeight: '280px' }"
      >
        <router-view />
      </a-layout-content>
<!--      <a-layout-footer style="text-align: center">-->
<!--        Ant Design ©2018 Created by Ant UED-->
<!--      </a-layout-footer>-->
    </a-layout>
  </a-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import {
  BookOutlined,
  FolderOutlined,
  FolderOpenOutlined,
  FileTextOutlined,
  MenuUnfoldOutlined,
  MenuFoldOutlined
} from '@ant-design/icons-vue'
import type { MenuClickEventHandler } from 'ant-design-vue/es/menu/src/interface'

const router = useRouter()
const collapsed = ref(false)
const selectedKeys = ref(['group'])
const openKeys = ref(['doc-center'])

const handleMenuClick: MenuClickEventHandler = ({ key }) => {
  router.push(`/home/${key}`)
}
</script>

<style scoped>
.trigger {
  font-size: 18px;
  line-height: 64px;
  padding: 0 24px;
  cursor: pointer;
  transition: color 0.3s;
}

.trigger:hover {
  color: #1890ff;
}

.logo {
  height: 32px;
  margin: 16px;
  background: rgba(255, 255, 255, 0.3);
}

.site-layout .site-layout-background {
  background: #fff;
}
[data-theme='dark'] .site-layout .site-layout-background {
  background: #141414;
}
</style>