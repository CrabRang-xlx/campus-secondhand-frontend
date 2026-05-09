<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'

const products = ref([])
const loading = ref(false)
const errorMessage = ref('')
const keyword = ref('')

const selectedProduct = ref(null)
const comments = ref([])
const detailLoading = ref(false)

const newCommentContent = ref('')
const commentSubmitting = ref(false)
const commentMessage = ref('')

const currentUser = ref(null)
const loginForm = ref({
  username: '',
  password: ''
})
const loginMessage = ref('')
const loginSubmitting = ref(false)

const loadCurrentUser = () => {
  const savedUser = localStorage.getItem('currentUser')
  if (savedUser) {
    currentUser.value = JSON.parse(savedUser)
  }
}

const login = async () => {
  if (!loginForm.value.username.trim()) {
    loginMessage.value = '请输入用户名'
    return
  }

  if (!loginForm.value.password.trim()) {
    loginMessage.value = '请输入密码'
    return
  }

  loginSubmitting.value = true
  loginMessage.value = ''

  try {
    const response = await axios.post('http://localhost:8080/api/users/login', {
      username: loginForm.value.username,
      password: loginForm.value.password
    })

    if (response.data.code === 200) {
      currentUser.value = response.data.data
      localStorage.setItem('currentUser', JSON.stringify(response.data.data))
      loginMessage.value = '登录成功'
      loginForm.value.password = ''
    } else {
      loginMessage.value = response.data.message || '登录失败'
    }
  } catch (error) {
    loginMessage.value = '登录失败，请确认后端服务已启动'
  } finally {
    loginSubmitting.value = false
  }
}

const logout = () => {
  currentUser.value = null
  localStorage.removeItem('currentUser')
  loginMessage.value = ''
  commentMessage.value = ''
}

const loadProducts = async () => {
  loading.value = true
  errorMessage.value = ''
  selectedProduct.value = null

  try {
    const response = await axios.get('http://localhost:8080/api/products')

    if (response.data.code === 200) {
      products.value = response.data.data
    } else {
      errorMessage.value = response.data.message || '商品加载失败'
    }
  } catch (error) {
    errorMessage.value = '无法连接后端服务，请确认 Spring Boot 已启动'
  } finally {
    loading.value = false
  }
}

const searchProducts = async () => {
  const searchText = keyword.value.trim()

  if (!searchText) {
    loadProducts()
    return
  }

  loading.value = true
  errorMessage.value = ''
  selectedProduct.value = null

  try {
    const response = await axios.get(
        `http://localhost:8080/api/products/search?keyword=${encodeURIComponent(searchText)}`
    )

    if (response.data.code === 200) {
      products.value = response.data.data
    } else {
      errorMessage.value = response.data.message || '搜索失败'
    }
  } catch (error) {
    errorMessage.value = '搜索失败，请确认后端服务已启动'
  } finally {
    loading.value = false
  }
}

const resetSearch = () => {
  keyword.value = ''
  loadProducts()
}

const loadComments = async (productId) => {
  const response = await axios.get(`http://localhost:8080/api/comments/product/${productId}`)

  if (response.data.code === 200) {
    comments.value = response.data.data
  } else {
    comments.value = []
  }
}

const openProductDetail = async (productId) => {
  detailLoading.value = true
  errorMessage.value = ''
  commentMessage.value = ''
  newCommentContent.value = ''

  try {
    const productResponse = await axios.get(`http://localhost:8080/api/products/${productId}`)

    if (productResponse.data.code === 200) {
      selectedProduct.value = productResponse.data.data
    } else {
      errorMessage.value = productResponse.data.message || '商品详情加载失败'
      return
    }

    await loadComments(productId)
  } catch (error) {
    errorMessage.value = '商品详情加载失败，请确认后端服务已启动'
  } finally {
    detailLoading.value = false
  }
}

const submitComment = async () => {
  if (!currentUser.value) {
    commentMessage.value = '请先登录后再留言'
    return
  }

  const content = newCommentContent.value.trim()

  if (!selectedProduct.value) {
    commentMessage.value = '请先选择商品'
    return
  }

  if (!content) {
    commentMessage.value = '留言内容不能为空'
    return
  }

  commentSubmitting.value = true
  commentMessage.value = ''

  try {
    const response = await axios.post('http://localhost:8080/api/comments', {
      productId: selectedProduct.value.id,
      userId: currentUser.value.id,
      content
    })

    if (response.data.code === 200) {
      newCommentContent.value = ''
      commentMessage.value = '留言发布成功'
      await loadComments(selectedProduct.value.id)
    } else {
      commentMessage.value = response.data.message || '留言发布失败'
    }
  } catch (error) {
    commentMessage.value = '留言发布失败，请确认后端服务已启动'
  } finally {
    commentSubmitting.value = false
  }
}

const backToList = () => {
  selectedProduct.value = null
  comments.value = []
  commentMessage.value = ''
  newCommentContent.value = ''
}

onMounted(() => {
  loadCurrentUser()
  loadProducts()
})
</script>

<template>
  <div class="app">
    <header class="header">
      <div class="logo">校园二手交易平台</div>

      <nav class="nav">
        <a href="#" @click.prevent="backToList">首页</a>
        <a href="#">发布商品</a>
        <a href="#">我的发布</a>
        <a href="#">我的收藏</a>

        <span v-if="currentUser" class="user-info">
          {{ currentUser.nickname || currentUser.username }}
        </span>

        <button v-if="currentUser" class="nav-button" @click="logout">
          退出
        </button>
      </nav>
    </header>

    <main class="main">
      <section class="hero">
        <h1>让校园闲置物品重新流动起来</h1>
        <p>面向学生的二手物品发布、搜索、收藏与留言平台。</p>

        <div v-if="!selectedProduct" class="search-box">
          <input
              v-model="keyword"
              type="text"
              placeholder="搜索商品名称、描述或分类"
              @keyup.enter="searchProducts"
          />
          <button @click="searchProducts">搜索</button>
          <button class="secondary" @click="resetSearch">重置</button>
        </div>
      </section>

      <section v-if="!currentUser && !selectedProduct" class="login-panel">
        <h2>用户登录</h2>

        <div class="login-form">
          <input
              v-model="loginForm.username"
              type="text"
              placeholder="用户名"
          />

          <input
              v-model="loginForm.password"
              type="password"
              placeholder="密码"
              @keyup.enter="login"
          />

          <button :disabled="loginSubmitting" @click="login">
            {{ loginSubmitting ? '登录中...' : '登录' }}
          </button>
        </div>

        <p v-if="loginMessage" class="login-message">
          {{ loginMessage }}
        </p>
      </section>

      <section v-if="selectedProduct" class="detail-section">
        <button class="back-button" @click="backToList">返回商品列表</button>

        <div v-if="detailLoading" class="message">
          正在加载详情...
        </div>

        <div v-else>
          <div class="detail-card">
            <div class="detail-image">
              {{ selectedProduct.category || '商品' }}
            </div>

            <div class="detail-info">
              <h2>{{ selectedProduct.title }}</h2>

              <div class="detail-price">
                ￥{{ selectedProduct.price }}
              </div>

              <p class="detail-description">
                {{ selectedProduct.description || '暂无描述' }}
              </p>

              <div class="detail-meta">
                <span>分类：{{ selectedProduct.category || '其他' }}</span>
                <span>状态：{{ selectedProduct.status }}</span>
                <span>商品 ID：{{ selectedProduct.id }}</span>
                <span>发布者 ID：{{ selectedProduct.userId }}</span>
              </div>
            </div>
          </div>

          <div class="comment-section">
            <h3>商品留言</h3>

            <div class="comment-form">
              <textarea
                  v-model="newCommentContent"
                  placeholder="输入留言，例如：你好，这个商品还在吗？"
              ></textarea>

              <div class="comment-actions">
                <span v-if="currentUser">
                  当前用户：{{ currentUser.nickname || currentUser.username }}
                </span>
                <span v-else>
                  请先登录后留言
                </span>

                <button :disabled="commentSubmitting" @click="submitComment">
                  {{ commentSubmitting ? '发布中...' : '发布留言' }}
                </button>
              </div>

              <div v-if="commentMessage" class="comment-message">
                {{ commentMessage }}
              </div>
            </div>

            <div v-if="comments.length === 0" class="message">
              暂无留言
            </div>

            <div v-else class="comment-list">
              <div
                  v-for="comment in comments"
                  :key="comment.id"
                  class="comment-item"
              >
                <div class="comment-header">
                  <strong>{{ comment.nickname || ('用户 ' + comment.userId) }}</strong>
                  <span>{{ comment.createTime }}</span>
                </div>
                <p>{{ comment.content }}</p>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section v-else class="products">
        <div class="section-title">
          <h2>商品列表</h2>
          <span>数据来自后端 MySQL</span>
        </div>

        <div v-if="loading" class="message">
          正在加载商品...
        </div>

        <div v-else-if="errorMessage" class="error">
          {{ errorMessage }}
        </div>

        <div v-else-if="products.length === 0" class="message">
          没有找到相关商品
        </div>

        <div v-else class="product-grid">
          <div
              v-for="product in products"
              :key="product.id"
              class="product-card"
              @click="openProductDetail(product.id)"
          >
            <div class="image-placeholder">
              {{ product.category || '商品' }}
            </div>

            <h3>{{ product.title }}</h3>

            <p>{{ product.description || '暂无描述' }}</p>

            <div class="meta">
              <span>{{ product.status }}</span>
              <span>ID: {{ product.id }}</span>
            </div>

            <div class="price">
              ￥{{ product.price }}
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  background: #f5f7fb;
  color: #1f2937;
  font-family: Arial, "Microsoft YaHei", sans-serif;
}

.header {
  height: 64px;
  padding: 0 48px;
  background: #ffffff;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #e5e7eb;
}

.logo {
  font-size: 22px;
  font-weight: 700;
  color: #2563eb;
}

.nav {
  display: flex;
  gap: 24px;
  align-items: center;
}

.nav a {
  color: #374151;
  text-decoration: none;
  font-size: 15px;
}

.nav a:hover {
  color: #2563eb;
}

.user-info {
  color: #2563eb;
  font-size: 14px;
  font-weight: 700;
}

.nav-button {
  border: none;
  background: #2563eb;
  color: white;
  padding: 6px 14px;
  border-radius: 999px;
  cursor: pointer;
}

.main {
  max-width: 1100px;
  margin: 0 auto;
  padding: 48px 24px;
}

.hero {
  background: linear-gradient(135deg, #2563eb, #38bdf8);
  color: white;
  padding: 56px;
  border-radius: 20px;
  text-align: center;
}

.hero h1 {
  font-size: 40px;
  margin-bottom: 16px;
}

.hero p {
  font-size: 18px;
  margin-bottom: 28px;
}

.search-box {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-top: 28px;
}

.search-box input {
  width: 360px;
  padding: 12px 16px;
  border: none;
  border-radius: 999px;
  font-size: 15px;
  outline: none;
}

.search-box button {
  border: none;
  background: white;
  color: #2563eb;
  padding: 12px 24px;
  border-radius: 999px;
  font-size: 15px;
  cursor: pointer;
}

.search-box .secondary {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.8);
}

.login-panel {
  margin-top: 32px;
  background: white;
  border-radius: 20px;
  padding: 28px;
  border: 1px solid #e5e7eb;
}

.login-panel h2 {
  margin-top: 0;
  font-size: 24px;
}

.login-form {
  display: flex;
  gap: 12px;
  align-items: center;
}

.login-form input {
  padding: 12px 14px;
  border: 1px solid #d1d5db;
  border-radius: 12px;
  font-size: 15px;
}

.login-form button {
  border: none;
  background: #2563eb;
  color: white;
  padding: 12px 24px;
  border-radius: 12px;
  cursor: pointer;
}

.login-form button:disabled {
  background: #9ca3af;
  cursor: not-allowed;
}

.login-message {
  margin-bottom: 0;
  color: #2563eb;
  font-size: 14px;
}

.products {
  margin-top: 48px;
}

.section-title {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 24px;
}

.section-title h2 {
  font-size: 28px;
  margin: 0;
}

.section-title span {
  color: #6b7280;
  font-size: 14px;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

.product-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  border: 1px solid #e5e7eb;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.product-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px rgba(15, 23, 42, 0.12);
}

.image-placeholder {
  height: 150px;
  border-radius: 12px;
  background: #e0ecff;
  color: #2563eb;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 16px;
  font-weight: 700;
}

.product-card h3 {
  margin: 0 0 8px;
  font-size: 20px;
}

.product-card p {
  color: #6b7280;
  font-size: 14px;
  min-height: 40px;
}

.meta {
  margin-top: 12px;
  display: flex;
  justify-content: space-between;
  color: #6b7280;
  font-size: 13px;
}

.price {
  margin-top: 12px;
  font-size: 20px;
  font-weight: 700;
  color: #ef4444;
}

.message {
  padding: 32px;
  background: white;
  border-radius: 16px;
  color: #6b7280;
  text-align: center;
}

.error {
  padding: 32px;
  background: #fff1f2;
  border: 1px solid #fecdd3;
  color: #be123c;
  border-radius: 16px;
  text-align: center;
}

.detail-section {
  margin-top: 48px;
}

.back-button {
  margin-bottom: 20px;
  border: none;
  background: #2563eb;
  color: white;
  padding: 10px 20px;
  border-radius: 999px;
  cursor: pointer;
}

.detail-card {
  display: grid;
  grid-template-columns: 360px 1fr;
  gap: 32px;
  background: white;
  border-radius: 20px;
  padding: 28px;
  border: 1px solid #e5e7eb;
}

.detail-image {
  height: 260px;
  border-radius: 16px;
  background: #e0ecff;
  color: #2563eb;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  font-weight: 700;
}

.detail-info h2 {
  font-size: 32px;
  margin: 0 0 16px;
}

.detail-price {
  font-size: 30px;
  color: #ef4444;
  font-weight: 700;
  margin-bottom: 20px;
}

.detail-description {
  color: #4b5563;
  line-height: 1.8;
}

.detail-meta {
  margin-top: 24px;
  display: grid;
  gap: 10px;
  color: #6b7280;
}

.comment-section {
  margin-top: 32px;
  background: white;
  border-radius: 20px;
  padding: 28px;
  border: 1px solid #e5e7eb;
}

.comment-section h3 {
  margin-top: 0;
  font-size: 24px;
}

.comment-form {
  margin-bottom: 24px;
  padding: 16px;
  background: #f9fafb;
  border-radius: 14px;
}

.comment-form textarea {
  width: 100%;
  min-height: 90px;
  resize: vertical;
  border: 1px solid #d1d5db;
  border-radius: 12px;
  padding: 12px;
  font-size: 15px;
  font-family: Arial, "Microsoft YaHei", sans-serif;
  box-sizing: border-box;
}

.comment-actions {
  margin-top: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #6b7280;
  font-size: 14px;
}

.comment-actions button {
  border: none;
  background: #2563eb;
  color: white;
  padding: 10px 18px;
  border-radius: 999px;
  cursor: pointer;
}

.comment-actions button:disabled {
  background: #9ca3af;
  cursor: not-allowed;
}

.comment-message {
  margin-top: 10px;
  color: #2563eb;
  font-size: 14px;
}

.comment-list {
  display: grid;
  gap: 16px;
}

.comment-item {
  padding: 16px;
  border-radius: 12px;
  background: #f9fafb;
}

.comment-header {
  display: flex;
  justify-content: space-between;
  color: #6b7280;
  font-size: 14px;
}

.comment-item p {
  margin-bottom: 0;
  color: #374151;
}
</style>