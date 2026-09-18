<template>
  <nav class="mobile-bottom-nav">
    <router-link
      v-for="tab in tabs"
      :key="tab.to"
      :to="tab.to"
      class="bottom-tab"
      :class="{ active: isActive(tab) }"
    >
      <span class="tab-icon">{{ tab.icon }}</span>
      <span class="tab-label">{{ tab.label }}</span>
      <span v-if="tab.badge && cartStore.itemCount" class="tab-badge">{{ cartStore.itemCount }}</span>
    </router-link>
  </nav>
</template>

<script setup>
import { useRoute } from 'vue-router'
import { useCartStore } from '../stores/cart'

const route = useRoute()
const cartStore = useCartStore()

const tabs = [
  { label: 'Trang chủ', to: '/', icon: '🏠', match: ['/'] },
  { label: 'Danh mục', to: '/categories', icon: '🗂️', match: ['/categories', '/products'] },
  { label: 'Nạp tiền', to: '/deposit', icon: '💳', match: ['/deposit'] },
  { label: 'Đơn hàng', to: '/orders', icon: '📦', match: ['/orders', '/cart', '/checkout'] },
  { label: 'Tài khoản', to: '/profile', icon: '👤', match: ['/profile', '/login', '/register'] },
]

const isActive = (tab) => tab.match.includes(route.path)
</script>

<style scoped>
.mobile-bottom-nav {
  display: none;
}

@media (max-width: 768px) {
  .mobile-bottom-nav {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 900;
    background: var(--bg-secondary);
    border-top: 1px solid var(--border);
    box-shadow: 0 -2px 12px rgba(0, 0, 0, 0.08);
    padding-bottom: env(safe-area-inset-bottom);
  }

  .bottom-tab {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2px;
    padding: 0.55rem 0.25rem;
    color: var(--text-secondary);
    font-size: 11px;
    font-weight: 600;
  }

  .bottom-tab.active {
    color: var(--primary);
  }

  .tab-icon {
    font-size: 1.25rem;
    line-height: 1;
  }

  .tab-badge {
    position: absolute;
    top: 2px;
    right: calc(50% - 22px);
    background: var(--danger);
    color: #fff;
    font-size: 10px;
    font-weight: 700;
    min-width: 16px;
    height: 16px;
    border-radius: 9999px;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0 4px;
  }
}
</style>
