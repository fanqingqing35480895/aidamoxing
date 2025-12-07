<script setup lang="ts">
import BasicLayout from '@/layouts/BasicLayout.vue'
</script>

<template>
  <BasicLayout />
</template>

<style>
/* 全局样式优化 */
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  font-family:
    -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell',
    'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  overflow-x: hidden;
  background: #000000;
  color: #ffffff;
}

html {
  overflow-x: hidden;
  background: #000000;
}

/* Ant Design 组件全局样式覆盖 */
:global(.ant-input),
:global(.ant-input-affix-wrapper) {
  background: rgba(0, 0, 0, 0.6) !important;
  border-color: rgba(0, 240, 255, 0.3) !important;
  color: #ffffff !important;
}

:global(.ant-input:focus),
:global(.ant-input-affix-wrapper:focus),
:global(.ant-input-affix-wrapper-focused) {
  border-color: #00f0ff !important;
  box-shadow: 0 0 20px rgba(0, 240, 255, 0.3) !important;
}

:global(.ant-input::placeholder) {
  color: rgba(0, 212, 255, 0.5) !important;
}

:global(.ant-textarea) {
  background: rgba(0, 0, 0, 0.6) !important;
  border-color: rgba(0, 240, 255, 0.3) !important;
  color: #ffffff !important;
}

:global(.ant-textarea:focus) {
  border-color: #00f0ff !important;
  box-shadow: 0 0 20px rgba(0, 240, 255, 0.3) !important;
}

:global(.ant-textarea::placeholder) {
  color: rgba(0, 212, 255, 0.5) !important;
}
</style>
