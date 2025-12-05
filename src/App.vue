<script setup lang="ts">
import BasicLayout from '@/layouts/BasicLayout.vue'
/**
 * 获取登录用户信息
 */
import { useLoginUserStore } from '@/stores/loginUser.ts'

const loginUserStore = useLoginUserStore()
loginUserStore.fetchLoginUser()

</script>

<template>
  <BasicLayout />
</template>

<style>
</style>

