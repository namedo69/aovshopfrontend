<template>
  <div class="home">


    <!-- Hero Section: full-width banner (template style) -->
    <section class="hero-wrapper">
      <div class="container">
        <!-- Banner Slider -->
        <div class="hero-banner">
          <div class="slider-container" v-if="bannerImages.length > 0">
            <div 
              class="slider-track" 
              :class="{ 'no-transition': isResetting }"
              :style="{ transform: `translateX(-${currentSlide * 100}%)` }"
            >
              <!-- Original slides + 1 cloned first slide for seamless loop -->
              <div 
                v-for="(banner, idx) in extendedBannerImages" 
                :key="idx" 
                class="slider-slide"
                :style="{ backgroundImage: `url(${banner})` }"
              ></div>
            </div>
          </div>
          <div v-else class="slider-fallback"></div>
          
          <!-- Slider dots -->
          <div v-if="bannerImages.length > 1" class="slider-dots">
            <button 
              v-for="(_, idx) in bannerImages" 
              :key="idx" 
              class="slider-dot" 
              :class="{ active: currentSlide === idx }"
              @click="goToSlide(idx)"
            ></button>
          </div>
        </div>
      </div>
    </section>

    <!-- Top deposit + recharge promo -->
    <section class="section sub-banner-row">
      <div class="container sub-banner-grid">
        <div class="sub-banner-deposit">
          <TopDepositLeaderboard />
        </div>
        <router-link to="/deposit" class="promo-card">
          <div class="promo-icon">⚡</div>
          <div>
            <p class="promo-title">Nạp thẻ & ngân hàng tự động 24/7</p>
            <p class="promo-text">Internet Banking, Ví điện tử quét QR — cộng tiền tức thì, miễn phí giao dịch.</p>
          </div>
          <span class="promo-arrow">→</span>
        </router-link>
      </div>
    </section>

    <!-- Categories Carousel -->
    <section class="section" id="categories-section">
      <div class="container">
        <div class="section-head reveal">
          <h2 class="section-title">Danh mục sản phẩm</h2>
          <router-link to="/categories" class="view-all-link">Xem tất cả →</router-link>
        </div>
        <div v-if="loading" class="skeleton-grid">
          <div v-for="n in 3" :key="'skel-cat-' + n" class="skeleton-category-card">
            <div class="skeleton skeleton-icon"></div>
            <div class="skeleton skeleton-title"></div>
            <div class="skeleton skeleton-text"></div>
          </div>
        </div>
        <div v-else-if="categories.length > 0" class="carousel-wrapper">
          <button class="carousel-arrow carousel-arrow-left" @click="prevCategorySlide" v-if="categories.length > categoriesPerView">‹</button>
          <div class="carousel-container" ref="categoryCarousel">
            <div 
              class="carousel-track category-carousel-track"
              :class="{ 'no-transition': categoryResetting }"
              :style="{ transform: `translateX(-${categorySlide * (100 / categoriesPerView)}%)` }"
            >
              <div 
                v-for="(category, idx) in extendedCategories" 
                :key="'cat-' + idx"
                class="carousel-item category-slide"
                :style="{ width: `${100 / categoriesPerView}%` }"
              >
                <router-link 
                  :to="`/products?category=${category.id}`"
                  class="category-card"
                >
                  <div class="category-thumb">
                    <img
                      v-if="category.image"
                      :src="getImageUrl(category.image)"
                      :alt="category.name"
                      loading="lazy"
                    />
                    <span v-else class="category-thumb-fallback">🎮</span>
                  </div>
                  <div class="category-content">
                    <h3>{{ category.name }}</h3>
                    <p>{{ category.products_count }} sản phẩm</p>
                  </div>
                </router-link>
              </div>
            </div>
          </div>
          <button class="carousel-arrow carousel-arrow-right" @click="nextCategorySlide" v-if="categories.length > categoriesPerView">›</button>
        </div>
      </div>
    </section>

    <!-- Guest auth prompt (template style) -->
    <section v-if="!authStore.isAuthenticated" class="auth-prompt-section">
      <div class="container">
        <div class="auth-prompt-card">
          <p class="auth-prompt-text">
            <strong>Xin chào Khách!</strong>
            Vui lòng đăng nhập để mua hàng và xem số dư
          </p>
          <div class="auth-prompt-actions">
            <router-link to="/login" class="btn btn-primary">Đăng Nhập</router-link>
            <router-link to="/register" class="btn btn-secondary">Đăng Ký</router-link>
          </div>
        </div>
      </div>
    </section>

    <!-- Recent orders marquee (template purchase-notice bar style) -->
    <section v-if="recentOrders.length > 0" class="recent-orders-bar">
      <div class="container">
        <div class="recent-orders-marquee">
          <span class="recent-badge">Mới</span>
          <div class="marquee-viewport">
            <div class="marquee-content">
              <span v-for="(order, idx) in loopedOrders" :key="idx" class="marquee-item">
                <strong class="marquee-user">{{ order.user }}</strong>&nbsp;vừa mua&nbsp;<strong>{{ order.product }}</strong>
                <span class="marquee-sep">•</span>
              </span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Featured Products Carousel -->
    <section class="section" v-if="featuredProducts.length">
      <div class="container">
        <div class="section-head reveal">
          <h2 class="section-title">Đang giảm giá</h2>
          <router-link to="/products" class="view-all-link">Xem tất cả →</router-link>
        </div>
        <div class="carousel-wrapper">
          <button class="carousel-arrow carousel-arrow-left" @click="prevFeaturedSlide" v-if="featuredProducts.length > productsPerView">‹</button>
          <div class="carousel-container">
            <div 
              class="carousel-track"
              :class="{ 'no-transition': featuredResetting }"
              :style="{ transform: `translateX(-${featuredSlide * (100 / productsPerView)}%)` }"
            >
              <div 
                v-for="(product, idx) in extendedFeaturedProducts" 
                :key="'featured-' + idx"
                class="carousel-item"
                :style="{ width: `${100 / productsPerView}%` }"
              >
                <ProductCard :product="product" />
              </div>
            </div>
          </div>
          <button class="carousel-arrow carousel-arrow-right" @click="nextFeaturedSlide" v-if="featuredProducts.length > productsPerView">›</button>
        </div>
      </div>
    </section>

    <!-- New Products Carousel -->
    <section class="section" v-if="newProducts.length">
      <div class="container">
        <div class="section-head reveal">
          <h2 class="section-title">Sản phẩm mới</h2>
          <router-link to="/products" class="view-all-link">Xem tất cả →</router-link>
        </div>
        <div class="carousel-wrapper">
          <button class="carousel-arrow carousel-arrow-left" @click="prevNewSlide" v-if="newProducts.length > productsPerView">‹</button>
          <div class="carousel-container">
            <div 
              class="carousel-track"
              :class="{ 'no-transition': newResetting }"
              :style="{ transform: `translateX(-${newSlide * (100 / productsPerView)}%)` }"
            >
              <div 
                v-for="(product, idx) in extendedNewProducts" 
                :key="'new-' + idx"
                class="carousel-item"
                :style="{ width: `${100 / productsPerView}%` }"
              >
                <ProductCard :product="product" />
              </div>
            </div>
          </div>
          <button class="carousel-arrow carousel-arrow-right" @click="nextNewSlide" v-if="newProducts.length > productsPerView">›</button>
        </div>
      </div>
    </section>

    <!-- Category product sections (template style: title + grid + view all) -->
    <section
      v-for="section in categorySections"
      :key="section.id"
      :id="`home-category-${section.id}`"
      class="section"
    >
      <div class="container">
        <div class="section-head reveal visible">
          <h2 class="section-title">{{ section.name }}</h2>
          <router-link :to="`/products?category=${section.id}`" class="view-all-link">Xem tất cả →</router-link>
        </div>
        <div class="shop-grid">
          <ProductCard v-for="product in section.products" :key="product.id" :product="product" />
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'
import { shopApi } from '../api'
import api from '../api'
import ProductCard from '../components/ProductCard.vue'
import TopDepositLeaderboard from '../components/TopDepositLeaderboard.vue'
import { getImageUrl } from '../utils/image'
import { useSettingsStore } from '../stores/settings'
import { useAuthStore } from '../stores/auth'

const settingsStore = useSettingsStore()
const authStore = useAuthStore()

const loading = ref(true)
const categories = ref([])
const featuredProducts = ref([])
const newProducts = ref([])
const recentOrders = ref([])
const categorySections = ref([])

// Marquee items duplicated for a seamless infinite loop
const loopedOrders = computed(() => [...recentOrders.value, ...recentOrders.value])

// Banner Slider Logic
const currentSlide = ref(0)
const isResetting = ref(false) // Flag to disable transition during reset
let slideInterval = null

const bannerImages = computed(() => {
  const banner = settingsStore.shopBanner
  if (!banner) return []
  
  try {
    const parsed = JSON.parse(banner)
    if (Array.isArray(parsed)) {
      return parsed.filter(b => b && b.trim())
    }
  } catch {
    // Not JSON, treat as single URL
  }
  return banner.trim() ? [banner] : []
})

// Extended array: original slides + 1 cloned first slide for seamless loop
const extendedBannerImages = computed(() => {
  if (bannerImages.value.length <= 1) return bannerImages.value
  return [...bannerImages.value, bannerImages.value[0]]
})

const goToSlide = (index) => {
  currentSlide.value = index
  resetAutoSlide()
}

const nextSlide = () => {
  if (bannerImages.value.length <= 1) return
  
  currentSlide.value++
  
  // When we reach the cloned slide (last position), wait for transition then reset
  if (currentSlide.value >= bannerImages.value.length) {
    setTimeout(() => {
      isResetting.value = true // Disable transition
      currentSlide.value = 0 // Jump back to real first slide
      
      // Re-enable transition after the position is reset
      requestAnimationFrame(() => {
        requestAnimationFrame(() => {
          isResetting.value = false
        })
      })
    }, 600) // Match the CSS transition duration
  }
}

const stopAutoSlide = () => {
  if (slideInterval) {
    clearInterval(slideInterval)
    slideInterval = null
  }
}

const startAutoSlide = () => {
  stopAutoSlide() // Luôn clear trước khi start mới
  if (bannerImages.value.length > 1) {
    slideInterval = setInterval(nextSlide, 5000)
  }
}

const resetAutoSlide = () => {
  startAutoSlide()
}

// ================== CAROUSEL LOGIC ==================

// Responsive items per view
const categoriesPerView = ref(3)
const productsPerView = ref(4)

// Category carousel state
const categorySlide = ref(0)
const categoryResetting = ref(false)
let categoryInterval = null

// Featured products carousel state
const featuredSlide = ref(0)
const featuredResetting = ref(false)
let featuredInterval = null

// New products carousel state
const newSlide = ref(0)
const newResetting = ref(false)
let newInterval = null

// Extended arrays for seamless infinite loop
const extendedCategories = computed(() => {
  if (categories.value.length <= categoriesPerView.value) return categories.value
  return [...categories.value, ...categories.value.slice(0, categoriesPerView.value)]
})

const extendedFeaturedProducts = computed(() => {
  if (featuredProducts.value.length <= productsPerView.value) return featuredProducts.value
  return [...featuredProducts.value, ...featuredProducts.value.slice(0, productsPerView.value)]
})

const extendedNewProducts = computed(() => {
  if (newProducts.value.length <= productsPerView.value) return newProducts.value
  return [...newProducts.value, ...newProducts.value.slice(0, productsPerView.value)]
})

// Generic carousel navigation helper
const createCarouselNav = (slideRef, resettingRef, items, perView) => {
  const next = () => {
    if (items.value.length <= perView.value) return
    slideRef.value++
    
    if (slideRef.value >= items.value.length) {
      setTimeout(() => {
        resettingRef.value = true
        slideRef.value = 0
        requestAnimationFrame(() => {
          requestAnimationFrame(() => {
            resettingRef.value = false
          })
        })
      }, 500)
    }
  }
  
  const prev = () => {
    if (items.value.length <= perView.value) return
    if (slideRef.value <= 0) {
      resettingRef.value = true
      slideRef.value = items.value.length
      requestAnimationFrame(() => {
        requestAnimationFrame(() => {
          resettingRef.value = false
          slideRef.value--
        })
      })
    } else {
      slideRef.value--
    }
  }
  
  return { next, prev }
}

// Category carousel navigation
const { next: nextCategorySlide, prev: prevCategorySlide } = createCarouselNav(
  categorySlide, categoryResetting, categories, categoriesPerView
)

// Featured products carousel navigation
const { next: nextFeaturedSlide, prev: prevFeaturedSlide } = createCarouselNav(
  featuredSlide, featuredResetting, featuredProducts, productsPerView
)

// New products carousel navigation
const { next: nextNewSlide, prev: prevNewSlide } = createCarouselNav(
  newSlide, newResetting, newProducts, productsPerView
)

// Auto-scroll for all carousels (slower intervals)
const startCarouselAutoSlide = () => {
  if (categories.value.length > categoriesPerView.value) {
    categoryInterval = setInterval(nextCategorySlide, 6000)
  }
  if (featuredProducts.value.length > productsPerView.value) {
    featuredInterval = setInterval(nextFeaturedSlide, 7000)
  }
  if (newProducts.value.length > productsPerView.value) {
    newInterval = setInterval(nextNewSlide, 8000)
  }
}

const stopCarouselAutoSlide = () => {
  if (categoryInterval) clearInterval(categoryInterval)
  if (featuredInterval) clearInterval(featuredInterval)
  if (newInterval) clearInterval(newInterval)
}

// Update items per view on resize
const updateItemsPerView = () => {
  const width = window.innerWidth
  if (width < 480) {
    categoriesPerView.value = 1
    productsPerView.value = 1
  } else if (width < 768) {
    categoriesPerView.value = 2
    productsPerView.value = 2
  } else if (width < 1024) {
    categoriesPerView.value = 3
    productsPerView.value = 3
  } else {
    categoriesPerView.value = 3
    productsPerView.value = 4
  }
}

// Chỉ reset khi bannerImages thay đổi thực sự
watch(bannerImages, (newVal, oldVal) => {
  if (JSON.stringify(newVal) !== JSON.stringify(oldVal)) {
    currentSlide.value = 0
    resetAutoSlide()
  }
})

onMounted(async () => {
  // Luôn refresh settings để lấy banner mới nhất
  await settingsStore.refreshShopInfo()
  
  try {
    const [catRes, featuredRes, newRes] = await Promise.all([
      shopApi.getCategories(),
      shopApi.getFeaturedProducts(),
      shopApi.getNewProducts(),
    ])
    categories.value = catRes.data
    featuredProducts.value = featuredRes.data
    newProducts.value = newRes.data

    // Category sections (template style): top 3 categories with a few products each
    try {
      const topCats = catRes.data.slice(0, 3)
      const sectionResults = await Promise.all(
        topCats.map(async (cat) => {
          try {
            const res = await shopApi.getProducts({ category_id: cat.id, per_page: 5 })
            return { id: cat.id, name: cat.name, products: res.data.data || [] }
          } catch {
            return { id: cat.id, name: cat.name, products: [] }
          }
        })
      )
      categorySections.value = sectionResults.filter((s) => s.products.length > 0)
    } catch (error) {
      console.error('Failed to load category sections:', error)
    }
  } catch (error) {
    console.error('Failed to load home data:', error)
  } finally {
    loading.value = false
  }
  
  startAutoSlide()
  
  // Initialize carousels
  updateItemsPerView()
  window.addEventListener('resize', updateItemsPerView)
  
  // Start carousel auto-scroll after data is loaded
  setTimeout(() => {
    startCarouselAutoSlide()
  }, 1000)
  
  // Fetch recent orders for mobile inline section
  try {
    const response = await api.get('/shop/recent-orders')
    if (response.data && response.data.length > 0) {
      recentOrders.value = response.data.slice(0, 5) // Max 5 for marquee
    }
  } catch (error) {
    console.error('Failed to fetch recent orders:', error)
  }
  


  // Initialize scroll reveal
  nextTick(() => {
    initScrollReveal()
  })
})

onUnmounted(() => {
  stopAutoSlide()
  stopCarouselAutoSlide()
  window.removeEventListener('resize', updateItemsPerView)
})



// Scroll reveal initialization
const initScrollReveal = () => {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible')
      }
    })
  }, {
    threshold: 0.1,
    rootMargin: '0px 0px -50px 0px'
  })
  
  document.querySelectorAll('.reveal').forEach(el => {
    observer.observe(el)
  })
}
</script>

<style scoped>
/* ===== HERO: full-width banner (template style) ===== */
.hero-wrapper {
  padding: 1rem 0 0;
}

.hero-banner {
  position: relative;
  min-height: 350px;
  border-radius: var(--radius-lg);
  overflow: hidden;
  text-align: center;
}

@media (max-width: 900px) {
  .hero-banner {
    min-height: 280px;
  }
}

@media (max-width: 480px) {
  .hero-wrapper {
    padding: 0.5rem 0 0;
  }

  .hero-banner {
    min-height: 170px;
  }
}

/* Slider Container */
.slider-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.slider-track {
  display: flex;
  height: 100%;
  transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.slider-track.no-transition {
  transition: none;
}

/* ===== CAROUSEL STYLES ===== */
.carousel-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  gap: 10px;
}

.carousel-container {
  overflow: hidden;
  flex: 1;
  border-radius: var(--radius-lg);
}

.carousel-track {
  display: flex;
  transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.carousel-track.no-transition {
  transition: none;
}

.carousel-item {
  flex-shrink: 0;
  padding: 0 8px;
  box-sizing: border-box;
}

/* Category slide wrapper - holds the card with spacing */
.category-slide {
  padding: 0 10px;
}

.category-slide .category-card {
  display: block;
  height: 100%;
}

.carousel-arrow {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: 1px solid var(--border);
  background: var(--bg-secondary);
  color: var(--text);
  font-size: 24px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  z-index: 10;
  box-shadow: var(--shadow);
}

.carousel-arrow:hover {
  background: var(--primary);
  border-color: var(--primary);
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(184, 0, 0, 0.3);
}

.carousel-arrow:active {
  transform: scale(0.95);
}

.carousel-arrow-left {
  margin-right: 5px;
}

.carousel-arrow-right {
  margin-left: 5px;
}

.slider-slide {
  flex-shrink: 0;
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
}

.slider-fallback {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #b80000, #f58300);
}

/* Banner Slider Dots */
.slider-dots {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
  z-index: 10;
}

.slider-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 2px solid rgba(255, 255, 255, 0.9);
  background: rgba(255, 255, 255, 0.35);
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.45);
  cursor: pointer;
  transition: all 0.3s ease;
  padding: 0;
}

.slider-dot:hover {
  border-color: white;
  background: rgba(255, 255, 255, 0.7);
}

.slider-dot.active {
  background: var(--primary);
  border-color: white;
  transform: scale(1.2);
}

/* ===== SECTIONS ===== */
.section {
  padding: 1rem 0;
  animation: fadeInUp 0.6s ease-out;
}

.section-title {
  font-size: 1.25rem;
  font-weight: 800;
  margin-bottom: 0;
  color: var(--text);
  text-transform: uppercase;
  letter-spacing: 0.02em;
  text-align: left;
  position: relative;
  display: block;
}

.section-title::after {
  content: '';
  display: block;
  width: 60px;
  height: 3px;
  background: var(--primary);
  margin: 8px 0 0;
  border-radius: 2px;
}

/* Template-style section header: title left + "Xem tất cả" right */
.section-head {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.view-all-link {
  color: var(--primary);
  font-weight: 600;
  font-size: 0.9rem;
  white-space: nowrap;
}

.view-all-link:hover {
  text-decoration: underline;
}

/* ===== AUTH PROMPT (template style) ===== */
.auth-prompt-section {
  padding-top: 0.75rem;
}

.auth-prompt-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  flex-wrap: wrap;
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 0.875rem 1.25rem;
  box-shadow: var(--shadow);
}

.auth-prompt-text {
  margin: 0;
  color: var(--text-secondary);
  font-size: 0.95rem;
}

.auth-prompt-actions {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;
}

/* ===== SUB-BANNER ROW: deposit + promo ===== */
.sub-banner-row {
  padding-bottom: 0;
}

.sub-banner-grid {
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 1rem;
  align-items: stretch;
}

.sub-banner-deposit {
  min-width: 0;
}

.promo-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 1.25rem 1.5rem;
  color: var(--text);
  transition: all 0.3s ease;
  min-width: 0;
}

.promo-card:hover {
  border-color: var(--primary);
  box-shadow: 0 8px 24px rgba(184, 0, 0, 0.12);
  transform: translateY(-2px);
  color: var(--text);
}

.promo-icon {
  font-size: 2rem;
  flex-shrink: 0;
}

.promo-title {
  margin: 0 0 0.25rem;
  font-weight: 800;
  text-transform: uppercase;
  color: var(--primary);
  font-size: 1rem;
}

.promo-text {
  margin: 0;
  color: var(--text-muted);
  font-size: 0.9rem;
}

.promo-arrow {
  margin-left: auto;
  font-size: 1.5rem;
  color: var(--primary);
  flex-shrink: 0;
}

@media (max-width: 900px) {
  .sub-banner-grid {
    grid-template-columns: 1fr;
  }
}

/* ===== CATEGORY PRODUCT GRID (template: 5-col desktop, 2-col mobile) ===== */
.shop-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 0.75rem;
}

@media (max-width: 1024px) {
  .shop-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 640px) {
  .shop-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* ===== CATEGORY CARDS (template tile style) ===== */
.category-card {
  position: relative;
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
  font-size: 2.5rem;
}

.category-overlay {
  display: none;
}

.category-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(184, 0, 0, 0.15);
  border-color: var(--primary);
}

.category-content {
  padding: 0.75rem 1rem;
  text-align: center;
}

.category-card h3 {
  font-size: 0.95rem;
  margin-bottom: 0.25rem;
  font-weight: 700;
  color: var(--text);
  text-transform: uppercase;
}

.category-content p {
  font-size: 0.8rem;
  color: var(--text-muted);
  margin: 0;
}

/* Mobile Responsive */
@media (max-width: 768px) {
  .section {
    padding: 3rem 0;
  }

  .section-title {
    font-size: 1.5rem;
    margin-bottom: 2rem;
  }

  .category-card {
    padding: 1.5rem 1rem;
  }

  .category-card:hover {
    transform: translateY(-4px) scale(1.01);
  }

  .category-icon {
    height: 60px;
    font-size: 2.5rem;
  }

  .category-img {
    width: 60px;
    height: 60px;
  }

  .category-card h3 {
    font-size: 1.1rem;
  }

  .category-card p {
    font-size: 0.85rem;
  }
}

@media (max-width: 480px) {
  .section {
    padding: 1.5rem 0;
  }

  .section-title {
    font-size: 1.1rem;
  }

  .category-card {
    padding: 0.75rem;
  }

  .category-icon {
    height: 40px;
    font-size: 1.5rem;
  }

  .category-img {
    width: 40px;
    height: 40px;
  }

  .category-card h3 {
    font-size: 0.9rem;
  }

  /* Carousel mobile */
  .carousel-arrow {
    width: 32px;
    height: 32px;
    font-size: 18px;
  }

  .carousel-item {
    padding: 0 5px;
  }

  .carousel-track {
    margin: 0 -5px;
  }

  .carousel-wrapper {
    margin: 0 -10px;
  }

  .carousel-container {
    margin: 0 10px;
  }
}



/* ===== SKELETON LOADING ===== */
.skeleton-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

@media (max-width: 768px) {
  .skeleton-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .skeleton-grid {
    grid-template-columns: 1fr;
  }
}

.skeleton-category-card {
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.skeleton {
  background: linear-gradient(
    90deg,
    rgba(0, 0, 0, 0.04) 25%,
    rgba(0, 0, 0, 0.08) 50%,
    rgba(0, 0, 0, 0.04) 75%
  );
  background-size: 200px 100%;
  animation: skeleton-loading 1.5s ease-in-out infinite;
  border-radius: 6px;
}

.skeleton-icon {
  width: 80px;
  height: 80px;
  border-radius: 16px;
}

.skeleton-title {
  height: 20px;
  width: 70%;
}

.skeleton-text {
  height: 14px;
  width: 50%;
}

@keyframes skeleton-loading {
  0% { background-position: -200px 0; }
  100% { background-position: calc(200px + 100%) 0; }
}

/* ===== SCROLL REVEAL ===== */
.reveal {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}

.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}

/* ===== RECENT ORDERS (template purchase-notice bar) ===== */
.recent-orders-bar {
  padding: 0.75rem 0 0;
}

.recent-orders-marquee {
  display: flex;
  align-items: center;
  gap: 10px;
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 0.5rem 0.75rem;
  box-shadow: var(--shadow);
  overflow: hidden;
}

.recent-badge {
  flex-shrink: 0;
  background: var(--primary);
  color: #fff;
  font-size: 11px;
  font-weight: 800;
  text-transform: uppercase;
  border-radius: 9999px;
  padding: 0.2rem 0.6rem;
}

.marquee-viewport {
  flex: 1;
  overflow: hidden;
}

.marquee-viewport:hover .marquee-content {
  animation-play-state: paused;
}

.marquee-content {
  display: flex;
  gap: 20px;
  animation: marqueeScroll 30s linear infinite;
  white-space: nowrap;
  width: max-content;
}

.marquee-item {
  font-size: 13px;
  color: var(--text-secondary);
}

.marquee-item strong {
  color: var(--text);
}

.marquee-user {
  color: var(--primary) !important;
}

.marquee-sep {
  color: var(--text-muted);
  margin-left: 10px;
}

@keyframes marqueeScroll {
  0% { transform: translateX(0); }
  100% { transform: translateX(-50%); }
}


</style>
