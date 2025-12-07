<template>
  <a-layout-footer class="footer">
    <div class="footer-content">
      <p class="copyright">
        <a
          href="https://www.doubao.com"
          target="_blank"
          rel="noopener noreferrer"
          class="author-link"
        >
          (外来系统）豆包智能助手系统
        </a>
      </p>
    </div>
  </a-layout-footer>
</template>

<script setup lang="ts">
// 无需额外的响应式数据
</script>

<style scoped>
.footer {
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  text-align: center;
  padding: 20px;
  margin-top: 40px;
  border-top: 1px solid rgba(0, 240, 255, 0.3);
  box-shadow: 0 -2px 20px rgba(0, 240, 255, 0.1);
}

.copyright {
  margin: 0;
  color: rgba(0, 212, 255, 0.7);
  font-size: 14px;
}

.author-link {
  color: #00d4ff;
  text-decoration: none;
  transition: all 0.3s;
}

.author-link:hover {
  color: #00f0ff;
  text-shadow: 0 0 10px rgba(0, 240, 255, 0.8);
}
</style>
