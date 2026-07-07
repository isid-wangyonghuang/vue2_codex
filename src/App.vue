<template>
  <div class="app-shell">
    <aside class="sidebar">
      <div class="brand">
        <div class="brand-mark">{{ text.brandMark }}</div>
        <div>
          <p>{{ text.appName }}</p>
          <span>{{ text.appSubtitle }}</span>
        </div>
      </div>

      <nav class="side-nav" :aria-label="text.mainNav">
        <button
          v-for="item in navItems"
          :key="item.id"
          type="button"
          :class="{ active: currentView === item.id }"
          @click="currentView = item.id"
          :title="text.nav[item.id]"
        >
          <component :is="item.icon" :size="19" aria-hidden="true" />
          <span>{{ text.nav[item.id] }}</span>
        </button>
      </nav>

      <div class="sidebar-footer">
        <p>{{ text.currentCycle }}</p>
        <strong>{{ text.cycleName }}</strong>
      </div>
    </aside>

    <main class="workspace">
      <header class="topbar">
        <div>
          <p class="eyebrow">Product Desk</p>
          <h1>{{ pageTitle }}</h1>
        </div>
        <div class="topbar-actions">
          <label class="language-select">
            <Languages :size="18" aria-hidden="true" />
            <select v-model="language" :aria-label="text.language">
              <option v-for="option in languageOptions" :key="option.value" :value="option.value">
                {{ option.label }}
              </option>
            </select>
          </label>
          <button type="button" class="icon-button" :title="text.exportData" @click="exportProducts">
            <Download :size="19" aria-hidden="true" />
          </button>
          <button type="button" class="primary-button" @click="openCreateModal">
            <Plus :size="18" aria-hidden="true" />
            <span>{{ text.addProduct }}</span>
          </button>
        </div>
      </header>

      <section v-if="currentView === 'overview'" class="content-stack">
        <div class="stats-grid">
          <article v-for="stat in stats" :key="stat.label" class="stat-card">
            <div :class="['stat-icon', stat.tone]">
              <component :is="stat.icon" :size="21" aria-hidden="true" />
            </div>
            <p>{{ stat.label }}</p>
            <strong>{{ stat.value }}</strong>
            <span>{{ stat.note }}</span>
          </article>
        </div>

        <section class="panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.productList }}</h2>
              <p>{{ text.totalRecords(filteredProducts.length) }}</p>
            </div>
            <div class="filters">
              <label class="search-field">
                <Search :size="18" aria-hidden="true" />
                <input v-model.trim="query" type="search" :placeholder="text.searchPlaceholder" />
              </label>
              <select v-model="categoryFilter" :aria-label="text.categoryFilterLabel">
                <option value="all">{{ text.allCategories }}</option>
                <option v-for="category in categories" :key="category" :value="category">{{ category }}</option>
              </select>
              <select v-model="statusFilter" :aria-label="text.statusFilterLabel">
                <option value="all">{{ text.allStatuses }}</option>
                <option v-for="status in statusOptions" :key="status" :value="status">{{ text.status[status] }}</option>
              </select>
            </div>
          </div>

          <div class="table-wrap">
            <table>
              <thead>
                <tr>
                  <th>{{ text.table.product }}</th>
                  <th>{{ text.table.sku }}</th>
                  <th>{{ text.table.category }}</th>
                  <th>{{ text.table.price }}</th>
                  <th>{{ text.table.stock }}</th>
                  <th>{{ text.table.status }}</th>
                  <th>{{ text.table.supplier }}</th>
                  <th class="actions-col">{{ text.table.actions }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="product in filteredProducts" :key="product.id">
                  <td>
                    <div class="product-cell">
                      <div class="avatar">{{ product.name.slice(0, 1) }}</div>
                      <div>
                        <strong>{{ product.name }}</strong>
                        <span>{{ text.sales(product.sales) }} · {{ product.unit }}</span>
                      </div>
                    </div>
                  </td>
                  <td>{{ product.sku }}</td>
                  <td>{{ product.category }}</td>
                  <td>{{ formatPrice(product.price) }}</td>
                  <td>
                    <div class="score">
                      <span>{{ product.stock }}</span>
                      <div class="meter"><i :style="{ width: stockPercent(product.stock) + '%' }"></i></div>
                    </div>
                  </td>
                  <td><span :class="['status-pill', statusTone(product.status)]">{{ text.status[product.status] }}</span></td>
                  <td>{{ product.supplier }}</td>
                  <td>
                    <div class="row-actions">
                      <button type="button" :title="text.edit" @click="openEditModal(product)">
                        <Pencil :size="17" aria-hidden="true" />
                      </button>
                      <button type="button" :title="text.delete" @click="removeProduct(product.id)">
                        <Trash2 :size="17" aria-hidden="true" />
                      </button>
                    </div>
                  </td>
                </tr>
                <tr v-if="filteredProducts.length === 0">
                  <td colspan="8" class="empty-state">{{ text.emptyState }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>
      </section>

      <section v-else-if="currentView === 'analytics'" class="content-grid">
        <article class="panel chart-panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.stockDistribution }}</h2>
              <p>{{ text.stockDistributionNote }}</p>
            </div>
          </div>
          <div class="bars">
            <div v-for="bucket in stockBuckets" :key="bucket.label" class="bar-row">
              <span>{{ bucket.label }}</span>
              <div class="bar-track"><i :style="{ width: bucket.percent + '%' }"></i></div>
              <strong>{{ bucket.count }}</strong>
            </div>
          </div>
        </article>

        <article class="panel ranking-panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.topSales }}</h2>
              <p>{{ text.topSalesNote }}</p>
            </div>
          </div>
          <div class="ranking-list">
            <div v-for="(product, index) in topProducts" :key="product.id" class="ranking-item">
              <span>{{ index + 1 }}</span>
              <div>
                <strong>{{ product.name }}</strong>
                <p>{{ product.category }}</p>
              </div>
              <em>{{ product.sales }}</em>
            </div>
          </div>
        </article>
      </section>

      <section v-else class="content-grid">
        <article class="panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.categoryOverview }}</h2>
              <p>{{ text.categoryOverviewNote }}</p>
            </div>
          </div>
          <div class="class-list">
            <div v-for="item in categorySummary" :key="item.category" class="class-item">
              <div>
                <strong>{{ item.category }}</strong>
                <span>{{ text.productCount(item.count) }}</span>
              </div>
              <p>{{ formatPrice(item.averagePrice) }}</p>
            </div>
          </div>
        </article>

        <article class="panel notice-panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.todoTitle }}</h2>
              <p>{{ text.todoNote }}</p>
            </div>
          </div>
          <div class="todo-list">
            <div v-for="todo in todos" :key="todo.title" class="todo-item">
              <component :is="todo.icon" :size="18" aria-hidden="true" />
              <div>
                <strong>{{ todo.title }}</strong>
                <span>{{ todo.desc }}</span>
              </div>
            </div>
          </div>
        </article>
      </section>
    </main>

    <div v-if="isModalOpen" class="modal-backdrop" @click.self="closeModal">
      <form class="product-modal" @submit.prevent="saveProduct">
        <div class="modal-header">
          <div>
            <p class="eyebrow">{{ editingId ? 'Edit Product' : 'New Product' }}</p>
            <h2>{{ editingId ? text.editProduct : text.addProduct }}</h2>
          </div>
          <button type="button" class="icon-button" :title="text.close" @click="closeModal">
            <X :size="19" aria-hidden="true" />
          </button>
        </div>

        <div class="form-grid">
          <label>
            {{ text.form.name }}
            <input v-model.trim="form.name" required />
          </label>
          <label>
            {{ text.form.sku }}
            <input v-model.trim="form.sku" required />
          </label>
          <label>
            {{ text.form.category }}
            <input v-model.trim="form.category" required />
          </label>
          <label>
            {{ text.form.unit }}
            <input v-model.trim="form.unit" required />
          </label>
          <label>
            {{ text.form.price }}
            <input v-model.number="form.price" type="number" min="0" step="0.01" required />
          </label>
          <label>
            {{ text.form.stock }}
            <input v-model.number="form.stock" type="number" min="0" required />
          </label>
          <label>
            {{ text.form.sales }}
            <input v-model.number="form.sales" type="number" min="0" required />
          </label>
          <label>
            {{ text.form.supplier }}
            <input v-model.trim="form.supplier" required />
          </label>
          <label class="full">
            {{ text.form.status }}
            <select v-model="form.status">
              <option v-for="status in statusOptions" :key="status" :value="status">{{ text.status[status] }}</option>
            </select>
          </label>
        </div>

        <div class="modal-actions">
          <button type="button" class="ghost-button" @click="closeModal">{{ text.cancel }}</button>
          <button type="submit" class="primary-button">
            <Save :size="18" aria-hidden="true" />
            <span>{{ text.save }}</span>
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { computed, reactive, ref, watch } from 'vue'
import {
  BarChart3,
  Boxes,
  ClipboardList,
  Download,
  Languages,
  LayoutDashboard,
  Package,
  Pencil,
  Plus,
  Save,
  Search,
  Tags,
  Trash2,
  TriangleAlert,
  X
} from '@lucide/vue'

const languageStorageKey = 'product-management-language'
const productStorageKey = 'product-management-data-i18n'
const statusOptions = ['active', 'lowStock', 'draft']

const translations = {
  zh: {
    documentTitle: '产品管理系统',
    brandMark: '品',
    appName: '产品管理系统',
    appSubtitle: '商品运营后台',
    mainNav: '主导航',
    currentCycle: '当前周期',
    cycleName: '2026 春季上新',
    language: '语言',
    exportData: '导出数据',
    addProduct: '新增产品',
    editProduct: '编辑产品',
    close: '关闭',
    cancel: '取消',
    save: '保存',
    edit: '编辑',
    delete: '删除',
    allCategories: '全部分类',
    allStatuses: '全部状态',
    categoryFilterLabel: '按分类筛选',
    statusFilterLabel: '按状态筛选',
    searchPlaceholder: '搜索产品名、SKU、供应商',
    productList: '产品列表',
    emptyState: '没有找到符合条件的产品',
    stockDistribution: '库存分布',
    stockDistributionNote: '按当前产品库存统计',
    topSales: '销量前五',
    topSalesNote: '快速查看热销产品',
    categoryOverview: '分类概览',
    categoryOverviewNote: '产品数量与平均价格',
    todoTitle: '运营提醒',
    todoNote: '来自库存、状态和销量数据',
    nav: {
      overview: '产品管理',
      analytics: '销售分析',
      categories: '分类信息'
    },
    table: {
      product: '产品',
      sku: 'SKU',
      category: '分类',
      price: '价格',
      stock: '库存',
      status: '状态',
      supplier: '供应商',
      actions: '操作'
    },
    form: {
      name: '产品名称',
      sku: 'SKU',
      category: '分类',
      unit: '单位',
      price: '价格',
      stock: '库存',
      sales: '销量',
      supplier: '供应商',
      status: '状态'
    },
    status: {
      active: '在售',
      lowStock: '低库存',
      draft: '草稿'
    },
    stats: {
      totalProducts: '产品总数',
      activeProducts: '在售产品',
      averagePrice: '平均价格',
      needsAttention: '需关注',
      categoryCount: count => `${count} 个分类`,
      normalStatus: '可正常销售',
      priceNote: '按当前产品计算',
      warningNote: '库存或状态异常'
    },
    todos: {
      lowStock: count => `${count} 个产品库存偏低`,
      lowStockDesc: '建议及时补货或调整库存预警',
      draft: count => `${count} 个产品仍是草稿`,
      draftDesc: '可编辑状态后发布到在售',
      hotSale: count => `${count} 个产品销量超过 800`,
      hotSaleDesc: '适合加入重点推广清单'
    },
    totalRecords: count => `共 ${count} 条记录`,
    productCount: count => `${count} 个产品`,
    sales: value => `销量 ${value}`,
    confirmDelete: name => `确定删除 ${name} 的记录吗？`,
    csvName: 'products-zh.csv',
    locale: 'zh-CN',
    currency: 'CNY'
  },
  ja: {
    documentTitle: '商品管理システム',
    brandMark: '品',
    appName: '商品管理システム',
    appSubtitle: '商品運営管理',
    mainNav: 'メインナビゲーション',
    currentCycle: '現在のサイクル',
    cycleName: '2026 春の新商品',
    language: '言語',
    exportData: 'データを書き出す',
    addProduct: '商品を追加',
    editProduct: '商品を編集',
    close: '閉じる',
    cancel: 'キャンセル',
    save: '保存',
    edit: '編集',
    delete: '削除',
    allCategories: 'すべてのカテゴリ',
    allStatuses: 'すべての状態',
    categoryFilterLabel: 'カテゴリで絞り込み',
    statusFilterLabel: '状態で絞り込み',
    searchPlaceholder: '商品名、SKU、仕入先で検索',
    productList: '商品一覧',
    emptyState: '条件に一致する商品が見つかりません',
    stockDistribution: '在庫分布',
    stockDistributionNote: '現在の商品在庫をもとに集計',
    topSales: '販売数トップ5',
    topSalesNote: '売れ筋商品をすばやく確認',
    categoryOverview: 'カテゴリ概要',
    categoryOverviewNote: '商品数と平均価格',
    todoTitle: '運営リマインド',
    todoNote: '在庫、状態、販売データから作成',
    nav: {
      overview: '商品管理',
      analytics: '販売分析',
      categories: 'カテゴリ情報'
    },
    table: {
      product: '商品',
      sku: 'SKU',
      category: 'カテゴリ',
      price: '価格',
      stock: '在庫',
      status: '状態',
      supplier: '仕入先',
      actions: '操作'
    },
    form: {
      name: '商品名',
      sku: 'SKU',
      category: 'カテゴリ',
      unit: '単位',
      price: '価格',
      stock: '在庫',
      sales: '販売数',
      supplier: '仕入先',
      status: '状態'
    },
    status: {
      active: '販売中',
      lowStock: '在庫少',
      draft: '下書き'
    },
    stats: {
      totalProducts: '商品総数',
      activeProducts: '販売中',
      averagePrice: '平均価格',
      needsAttention: '要確認',
      categoryCount: count => `${count} カテゴリ`,
      normalStatus: '販売可能',
      priceNote: '現在の商品から算出',
      warningNote: '在庫または状態に注意'
    },
    todos: {
      lowStock: count => `${count} 商品の在庫が少なめです`,
      lowStockDesc: '早めの補充または在庫基準の見直しをおすすめします',
      draft: count => `${count} 商品が下書きのままです`,
      draftDesc: '状態を編集して販売中にできます',
      hotSale: count => `${count} 商品が800件以上販売されています`,
      hotSaleDesc: '重点プロモーション候補になります'
    },
    totalRecords: count => `全 ${count} 件`,
    productCount: count => `${count} 商品`,
    sales: value => `販売数 ${value}`,
    confirmDelete: name => `${name} の記録を削除しますか？`,
    csvName: 'products-ja.csv',
    locale: 'ja-JP',
    currency: 'JPY'
  },
  en: {
    documentTitle: 'Product Management System',
    brandMark: 'P',
    appName: 'Product Management',
    appSubtitle: 'Merchandising Desk',
    mainNav: 'Main navigation',
    currentCycle: 'Current cycle',
    cycleName: 'Spring 2026 Launch',
    language: 'Language',
    exportData: 'Export data',
    addProduct: 'Add Product',
    editProduct: 'Edit Product',
    close: 'Close',
    cancel: 'Cancel',
    save: 'Save',
    edit: 'Edit',
    delete: 'Delete',
    allCategories: 'All categories',
    allStatuses: 'All statuses',
    categoryFilterLabel: 'Filter by category',
    statusFilterLabel: 'Filter by status',
    searchPlaceholder: 'Search product, SKU, or supplier',
    productList: 'Product List',
    emptyState: 'No products match the current filters',
    stockDistribution: 'Stock Distribution',
    stockDistributionNote: 'Calculated from current inventory',
    topSales: 'Top 5 Sales',
    topSalesNote: 'Quickly review best-selling products',
    categoryOverview: 'Category Overview',
    categoryOverviewNote: 'Product count and average price',
    todoTitle: 'Operations Follow-ups',
    todoNote: 'Generated from stock, status, and sales data',
    nav: {
      overview: 'Products',
      analytics: 'Sales Analytics',
      categories: 'Categories'
    },
    table: {
      product: 'Product',
      sku: 'SKU',
      category: 'Category',
      price: 'Price',
      stock: 'Stock',
      status: 'Status',
      supplier: 'Supplier',
      actions: 'Actions'
    },
    form: {
      name: 'Product Name',
      sku: 'SKU',
      category: 'Category',
      unit: 'Unit',
      price: 'Price',
      stock: 'Stock',
      sales: 'Sales',
      supplier: 'Supplier',
      status: 'Status'
    },
    status: {
      active: 'Active',
      lowStock: 'Low Stock',
      draft: 'Draft'
    },
    stats: {
      totalProducts: 'Total Products',
      activeProducts: 'Active',
      averagePrice: 'Average Price',
      needsAttention: 'Needs Review',
      categoryCount: count => `${count} categories`,
      normalStatus: 'Ready for sale',
      priceNote: 'Across current products',
      warningNote: 'Stock or status issue'
    },
    todos: {
      lowStock: count => `${count} products are low on stock`,
      lowStockDesc: 'Restock soon or adjust inventory thresholds',
      draft: count => `${count} products are still drafts`,
      draftDesc: 'Edit status to publish them',
      hotSale: count => `${count} products sold over 800 units`,
      hotSaleDesc: 'Good candidates for promotion'
    },
    totalRecords: count => `${count} records`,
    productCount: count => `${count} products`,
    sales: value => `${value} sold`,
    confirmDelete: name => `Delete ${name}'s record?`,
    csvName: 'products-en.csv',
    locale: 'en-US',
    currency: 'USD'
  }
}

const languageOptions = [
  { value: 'ja', label: '日本語' },
  { value: 'zh', label: '中文' },
  { value: 'en', label: 'English' }
]

const seedProducts = [
  { id: 1, name: 'スマートウォッチ S9', sku: 'PRD-2026-001', category: 'デジタル', unit: 'piece', price: 19800, stock: 86, sales: 1240, status: 'active', supplier: 'Tokyo Tech Supply' },
  { id: 2, name: 'ワイヤレスイヤホン AirLite', sku: 'PRD-2026-002', category: 'デジタル', unit: 'set', price: 12800, stock: 42, sales: 980, status: 'active', supplier: 'Sound Bridge' },
  { id: 3, name: 'ミニ加湿器 PureMist', sku: 'PRD-2026-003', category: '生活家電', unit: 'piece', price: 5600, stock: 18, sales: 460, status: 'lowStock', supplier: 'Home Lab' },
  { id: 4, name: '折りたたみデスク StandFit', sku: 'PRD-2026-004', category: 'オフィス', unit: 'piece', price: 16800, stock: 64, sales: 720, status: 'active', supplier: 'Work Plus' },
  { id: 5, name: '保温ボトル Thermo One', sku: 'PRD-2026-005', category: '生活雑貨', unit: 'piece', price: 3200, stock: 12, sales: 1180, status: 'lowStock', supplier: 'Daily Goods' },
  { id: 6, name: 'LEDデスクライト Nova', sku: 'PRD-2026-006', category: 'オフィス', unit: 'piece', price: 7600, stock: 52, sales: 530, status: 'active', supplier: 'Light Studio' },
  { id: 7, name: 'コットン収納ボックス', sku: 'PRD-2026-007', category: '生活雑貨', unit: 'box', price: 2400, stock: 95, sales: 330, status: 'draft', supplier: 'Daily Goods' },
  { id: 8, name: 'ポータブル電源 MiniVolt', sku: 'PRD-2026-008', category: 'アウトドア', unit: 'piece', price: 43800, stock: 27, sales: 860, status: 'active', supplier: 'Outdoor Base' }
]

const normalizeProduct = product => ({
  ...product,
  status: normalizeStatus(product.status)
})

const normalizeStatus = status => {
  if (['lowStock', '低库存', '在庫少', 'Low Stock'].includes(status)) return 'lowStock'
  if (['draft', '草稿', '下書き', 'Draft'].includes(status)) return 'draft'
  return 'active'
}

const loadLanguage = () => {
  const saved = localStorage.getItem(languageStorageKey)
  return translations[saved] ? saved : 'ja'
}

const loadProducts = () => {
  try {
    const saved = localStorage.getItem(productStorageKey)
    return saved ? JSON.parse(saved).map(normalizeProduct) : seedProducts
  } catch {
    return seedProducts
  }
}

const language = ref(loadLanguage())
const products = ref(loadProducts())
const currentView = ref('overview')
const query = ref('')
const categoryFilter = ref('all')
const statusFilter = ref('all')
const isModalOpen = ref(false)
const editingId = ref(null)

const emptyForm = () => ({
  name: '',
  sku: '',
  category: 'デジタル',
  unit: 'piece',
  price: 1000,
  stock: 50,
  sales: 0,
  supplier: '',
  status: 'active'
})

const form = reactive(emptyForm())
const text = computed(() => translations[language.value])

watch(language, value => {
  localStorage.setItem(languageStorageKey, value)
  document.documentElement.lang = value === 'zh' ? 'zh-CN' : value
  document.title = translations[value].documentTitle
}, { immediate: true })

watch(products, value => {
  localStorage.setItem(productStorageKey, JSON.stringify(value))
}, { deep: true })

const navItems = [
  { id: 'overview', icon: LayoutDashboard },
  { id: 'analytics', icon: BarChart3 },
  { id: 'categories', icon: Tags }
]

const pageTitle = computed(() => text.value.nav[currentView.value] || text.value.nav.overview)
const categories = computed(() => [...new Set(products.value.map(product => product.category))].sort())

const filteredProducts = computed(() => {
  const keyword = query.value.toLowerCase()
  return products.value.filter(product => {
    const matchQuery = [product.name, product.sku, product.supplier, product.category].some(value => String(value).toLowerCase().includes(keyword))
    const matchCategory = categoryFilter.value === 'all' || product.category === categoryFilter.value
    const matchStatus = statusFilter.value === 'all' || product.status === statusFilter.value
    return matchQuery && matchCategory && matchStatus
  })
})

const averagePrice = computed(() => {
  if (!products.value.length) return 0
  return Math.round(products.value.reduce((sum, product) => sum + Number(product.price), 0) / products.value.length)
})

const activeCount = computed(() => products.value.filter(product => product.status === 'active').length)
const attentionCount = computed(() => products.value.filter(product => product.stock < 25 || product.status !== 'active').length)

const stats = computed(() => [
  { label: text.value.stats.totalProducts, value: products.value.length, note: text.value.stats.categoryCount(categories.value.length), icon: Package, tone: 'blue' },
  { label: text.value.stats.activeProducts, value: activeCount.value, note: text.value.stats.normalStatus, icon: Boxes, tone: 'green' },
  { label: text.value.stats.averagePrice, value: formatPrice(averagePrice.value), note: text.value.stats.priceNote, icon: ClipboardList, tone: 'amber' },
  { label: text.value.stats.needsAttention, value: attentionCount.value, note: text.value.stats.warningNote, icon: TriangleAlert, tone: 'red' }
])

const topProducts = computed(() => [...products.value].sort((a, b) => b.sales - a.sales).slice(0, 5))

const stockBuckets = computed(() => {
  const buckets = [
    { label: '80+', min: 80, max: Number.POSITIVE_INFINITY },
    { label: '50-79', min: 50, max: 79 },
    { label: '25-49', min: 25, max: 49 },
    { label: '1-24', min: 1, max: 24 },
    { label: '0', min: 0, max: 0 }
  ]
  const total = products.value.length || 1
  return buckets.map(bucket => {
    const count = products.value.filter(product => product.stock >= bucket.min && product.stock <= bucket.max).length
    return { ...bucket, count, percent: Math.round((count / total) * 100) }
  })
})

const categorySummary = computed(() => categories.value.map(category => {
  const members = products.value.filter(product => product.category === category)
  const averagePrice = members.length
    ? Math.round(members.reduce((sum, product) => sum + Number(product.price), 0) / members.length)
    : 0
  return { category, count: members.length, averagePrice }
}))

const todos = computed(() => [
  {
    title: text.value.todos.lowStock(products.value.filter(product => product.stock < 25).length),
    desc: text.value.todos.lowStockDesc,
    icon: TriangleAlert
  },
  {
    title: text.value.todos.draft(products.value.filter(product => product.status === 'draft').length),
    desc: text.value.todos.draftDesc,
    icon: ClipboardList
  },
  {
    title: text.value.todos.hotSale(products.value.filter(product => product.sales >= 800).length),
    desc: text.value.todos.hotSaleDesc,
    icon: BarChart3
  }
])

const resetForm = values => {
  Object.assign(form, emptyForm(), values)
}

const openCreateModal = () => {
  editingId.value = null
  resetForm()
  isModalOpen.value = true
}

const openEditModal = product => {
  editingId.value = product.id
  resetForm(product)
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
}

const saveProduct = () => {
  const payload = {
    ...form,
    price: Math.max(0, Number(form.price)),
    stock: Math.max(0, Number(form.stock)),
    sales: Math.max(0, Number(form.sales))
  }

  if (editingId.value) {
    products.value = products.value.map(product => product.id === editingId.value ? { ...product, ...payload } : product)
  } else {
    products.value = [{ id: Date.now(), ...payload }, ...products.value]
  }

  closeModal()
}

const removeProduct = id => {
  const target = products.value.find(product => product.id === id)
  if (target && window.confirm(text.value.confirmDelete(target.name))) {
    products.value = products.value.filter(product => product.id !== id)
  }
}

const statusTone = status => {
  if (status === 'active') return 'ok'
  if (status === 'draft') return 'pending'
  return 'danger'
}

const stockPercent = stock => Math.min(100, Math.max(0, Math.round((Number(stock) / 100) * 100)))

const formatPrice = value => new Intl.NumberFormat(text.value.locale, {
  style: 'currency',
  currency: text.value.currency,
  maximumFractionDigits: 0
}).format(Number(value))

const exportProducts = () => {
  const header = [
    text.value.table.product,
    text.value.table.sku,
    text.value.table.category,
    text.value.table.price,
    text.value.table.stock,
    text.value.form.sales,
    text.value.table.status,
    text.value.table.supplier
  ]
  const rows = products.value.map(product => [
    product.name,
    product.sku,
    product.category,
    product.price,
    product.stock,
    product.sales,
    text.value.status[product.status],
    product.supplier
  ])
  const csv = [header, ...rows].map(row => row.map(cell => `"${String(cell).replaceAll('"', '""')}"`).join(',')).join('\n')
  const blob = new Blob([`\uFEFF${csv}`], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = text.value.csvName
  link.click()
  URL.revokeObjectURL(link.href)
}
</script>
