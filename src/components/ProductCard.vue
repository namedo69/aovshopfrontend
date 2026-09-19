<template>
  <router-link :to="`/products/${product.id}`" class="product-card">
    <div class="product-image-wrapper">
      <img 
        :src="getImageUrl(product.image)" 
        :alt="product.name"
        class="product-image"
        loading="lazy"
      />
    </div>
    
    <div class="product-info">
      <div class="product-category" v-if="product.category">
        {{ product.category.name }}
      </div>
      <h3 class="product-name">{{ product.name }}</h3>
      <div class="product-divider"></div>
      <div class="product-stats">
        <span class="product-stock" :class="stockClass">
          <span class="stock-dot"></span>
          {{ stockText }}
        </span>
        <span class="product-stats-sep">|</span>
        <span class="product-sold">
          Đã bán {{ product.sold_count || 0 }}
        </span>
      </div>
      <div class="product-pricing">
        <span class="product-price">{{ formatPrice(currentPrice) }}</span>
        <span v-if="isOnSale" class="product-price-old">{{ formatPrice(product.price) }}</span>
        <span v-if="isOnSale" class="discount-badge">-{{ discountPercent }}%</span>
      </div>
      <!-- Stock Progress Bar -->
      <div v-if="product.stock > 0 && product.stock <= 20" class="stock-bar-wrapper">
        <div class="stock-bar" :style="{ width: stockPercent + '%' }"></div>
        <span class="stock-text">Còn {{ product.stock }} sản phẩm</span>
      </div>
      <div class="product-action">
        <span class="btn-buy">MUA NGAY</span>
      </div>
    </div>
    
    <!-- Badges -->
    <div v-if="isOnSale" class="sale-badge">
      <span class="sale-text">FLASH SALE</span>
    </div>
    <div v-if="product.is_preorder" class="preorder-badge">ĐẶT TRƯỚC</div>
    <div v-else-if="isCheckpass" class="preorder-badge">KEY CHECKPASS</div>
    <div v-else-if="isNew" class="new-badge">MỚI</div>
    
    <!-- Quick View Modal -->
    <QuickViewModal 
      :show="showQuickView" 
      :product="product" 
      @close="showQuickView = false" 
    />
  </router-link>
</template>

<script setup>
import { computed, ref } from 'vue'
import { getImageUrl } from '../utils/image'
import QuickViewModal from './QuickViewModal.vue'

const props = defineProps({
  product: {
    type: Object,
    required: true
  }
})

const showQuickView = ref(false)

const currentPrice = computed(() => props.product.sale_price || props.product.price)
const isOnSale = computed(() => props.product.sale_price && props.product.sale_price < props.product.price)
const isNew = computed(() => {
  if (!props.product.created_at) return false
  const createdDate = new Date(props.product.created_at)
  const now = new Date()
  const diffDays = (now - createdDate) / (1000 * 60 * 60 * 24)
  return diffDays <= 7
})

const discountPercent = computed(() => {
  if (!isOnSale.value) return 0
  return Math.round((1 - props.product.sale_price / props.product.price) * 100)
})

const stockPercent = computed(() => {
  const maxStock = 50
  return Math.min((props.product.stock / maxStock) * 100, 100)
})

const stockClass = computed(() => ({
  'text-preorder': props.product.is_preorder,
  'text-success': isCheckpass.value || (!props.product.is_preorder && props.product.stock > 10),
  'text-warning': !props.product.is_preorder && !isCheckpass.value && props.product.stock > 0 && props.product.stock <= 10,
  'text-danger': !props.product.is_preorder && !isCheckpass.value && props.product.stock === 0,
}))

const isCheckpass = computed(() => Number(props.product.checkpass_hours ?? props.product.checkpassHours ?? 0) > 0)

const stockText = computed(() => {
  if (props.product.is_preorder) return 'Đặt trước'
  if (isCheckpass.value) return 'Key tự cấp'
  if (props.product.stock === 0) return 'Hết hàng'
  if (props.product.stock <= 10) return `Còn ${props.product.stock}`
  return 'Còn hàng'
})

const formatPrice = (price) => {
  return new Intl.NumberFormat('vi-VN', {
    style: 'currency',
    currency: 'VND'
  }).format(price)
}
</script>

<style scoped>
.product-card {
  position: relative;
  display: flex;
  flex-direction: column;
  height: 100%;
  color: var(--text);
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  overflow: hidden;
  transition: all 0.3s ease;
}

.product-card:hover {
  border-color: var(--primary);
  box-shadow: 0 8px 24px rgba(184, 0, 0, 0.12);
  transform: translateY(-3px);
}

/* Image Wrapper — chuẩn 5:3 / 16:9 của shopnhanviencu.net */
.product-image-wrapper {
  position: relative;
  overflow: hidden;
  width: 100%;
  aspect-ratio: 5 / 3;
  flex-shrink: 0;
  background: var(--bg-tertiary);
}

.product-image {
  width: 100%;
  height: 100%;
  aspect-ratio: 5 / 3;
  object-fit: cover;
  display: block;
  background: var(--bg-tertiary);
  transition: transform 0.4s ease;
}

.product-card:hover .product-image {
  transform: scale(1.05);
}

/* Product Info */
.product-info {
  padding: 0.85rem 1rem 1rem;
  position: relative;
  z-index: 1;
  background: var(--bg-secondary);
  display: flex;
  flex-direction: column;
  flex: 1;
}

.product-category {
  font-size: 0.7rem;
  color: var(--primary);
  margin-bottom: 0.3rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.product-name {
  font-weight: 600;
  margin-bottom: 0.4rem;
  color: var(--text);
  font-size: 0.95rem;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Divider */
.product-divider {
  height: 1px;
  background: var(--border);
  margin: 0.4rem 0;
}

/* Stats line: stock | sold */
.product-stats {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.8rem;
  color: var(--text-muted);
  margin-bottom: 0.4rem;
}

.product-stats-sep {
  color: var(--border);
}

/* Pricing */
.product-pricing {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
  flex-wrap: wrap;
}

.product-price {
  font-size: 1.15rem;
  font-weight: 700;
  color: var(--primary);
}

.product-price-old {
  font-size: 0.85rem;
  color: var(--text-muted);
  text-decoration: line-through;
}

.discount-badge {
  background: #fff0f0;
  color: var(--primary);
  padding: 1px 6px;
  border-radius: 4px;
  font-size: 0.7rem;
  font-weight: 700;
  border: 1px solid rgba(184, 0, 0, 0.2);
}

/* Stock Progress Bar */
.stock-bar-wrapper {
  margin-top: 6px;
  position: relative;
  height: 4px;
  background: rgba(0, 0, 0, 0.06);
  border-radius: 2px;
  overflow: hidden;
}

.stock-bar {
  height: 100%;
  background: linear-gradient(90deg, var(--danger), var(--warning));
  border-radius: 2px;
  transition: width 0.5s ease;
}

.stock-text {
  position: absolute;
  right: 0;
  top: -16px;
  font-size: 10px;
  color: var(--text-muted);
}

/* MUA NGAY Button */
.product-action {
  margin-top: auto;
  padding-top: 0.6rem;
}

.btn-buy {
  display: block;
  width: 100%;
  text-align: center;
  padding: 0.45rem;
  background: var(--primary);
  color: white;
  border-radius: var(--radius-sm);
  font-size: 0.85rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  transition: background 0.2s;
}

.product-card:hover .btn-buy {
  background: var(--primary-dark);
}

/* Badges */
.sale-badge {
  position: absolute;
  top: 10px;
  right: 10px;
  background: var(--primary);
  color: white;
  padding: 3px 10px;
  border-radius: 4px;
  font-size: 0.65rem;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  z-index: 2;
  display: flex;
  align-items: center;
}

.sale-text {
  position: relative;
}

.new-badge {
  position: absolute;
  top: 10px;
  left: 10px;
  background: var(--success);
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 4px;
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  z-index: 2;
}

.text-success { color: var(--success); }
.text-warning { color: var(--warning); }
.text-danger { color: var(--danger); }
.text-preorder { color: #d97706; font-weight: 600; }

.preorder-badge {
  position: absolute;
  top: 10px;
  left: 10px;
  background: #f58300;
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 4px;
  font-size: 0.65rem;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  z-index: 2;
}

/* Responsive */
@media (max-width: 640px) {
  .product-info {
    padding: 0.6rem 0.75rem;
  }
  
  .product-name {
    font-size: 0.85rem;
  }
  
  .product-price {
    font-size: 1rem;
  }

  .product-stats {
    font-size: 0.72rem;
  }
  
  .btn-buy {
    padding: 0.35rem;
    font-size: 0.8rem;
  }
}
</style>
