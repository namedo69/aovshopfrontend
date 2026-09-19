<template>
  <div class="categories-page">
    <div class="container">
      <div class="page-header">
        <h1 class="page-title">Danh mục sản phẩm</h1>
        <p class="page-subtitle">Khám phá các danh mục sản phẩm của chúng tôi</p>
      </div>

      <div v-if="loading" class="loading"><div class="spinner"></div></div>
      
      <div v-else-if="categories.length === 0" class="empty-state">
        <div class="empty-icon">📁</div>
        <h3 class="empty-title">Chưa có danh mục nào</h3>
        <p class="empty-text">Các danh mục sẽ được hiển thị ở đây</p>
      </div>

      <div v-else class="categories-grid">
        <router-link 
          v-for="cat in categories" 
          :key="cat.id" 
          :to="`/products?category=${cat.id}`"
          class="category-card"
        >
          <div class="category-thumb">
            <img
              v-if="cat.image"
              :src="cat.image"
              :alt="cat.name"
              loading="lazy"
            />
            <span v-else class="category-thumb-fallback">🎮</span>
          </div>
          <div class="category-content">
            <h3 class="category-name">{{ cat.name }}</h3>
            <p class="category-count">{{ cat.products_count || 0 }} sản phẩm</p>
          </div>
        </router-link>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { shopApi } from '../api'

const loading = ref(true)
const categories = ref([])

const loadCategories = async () => {
  loading.value = true
  try {
    const response = await shopApi.getCategories()
    categories.value = response.data
  } catch (error) {
    console.error('Failed to load categories:', error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  loadCategories()
})
</script>

<style scoped>
.categories-page {
  padding: 2rem 0;
  min-height: 60vh;
}

.categories-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
}

.category-card {
  display: block;
  border-radius: var(--radius-lg);
  overflow: hidden;
  text-decoration: none;
  color: var(--text);
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  transition: all 0.3s ease;
}

.category-thumb {
  aspect-ratio: 16 / 9;
  background: var(--bg-tertiary);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.category-thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.4s ease;
}

.category-card:hover .category-thumb img {
  transform: scale(1.05);
}

.category-thumb-fallback {
  font-size: 2rem;
}

.category-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 18px rgba(184, 0, 0, 0.12);
  border-color: var(--primary);
  color: var(--text);
}

.category-content {
  padding: 0.6rem 0.75rem;
  text-align: center;
}

.category-name {
  font-size: 0.85rem;
  font-weight: 700;
  margin-bottom: 0.15rem;
  text-transform: uppercase;
  color: var(--text);
}

.category-count {
  font-size: 0.75rem;
  color: var(--text-muted);
  margin: 0;
}

@media (max-width: 480px) {
  .categories-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 0.75rem;
  }

  .category-name {
    font-size: 0.85rem;
  }
}
</style>

