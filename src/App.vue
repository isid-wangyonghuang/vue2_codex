<template>
  <div class="app-shell">
    <aside class="sidebar">
      <div class="brand">
        <div class="brand-mark">車</div>
        <div>
          <p>中古車管理システム</p>
          <span>Spring Boot + MyBatis</span>
        </div>
      </div>

      <nav class="side-nav" aria-label="メインナビゲーション">
        <button
          v-for="item in navItems"
          :key="item.id"
          type="button"
          :class="{ active: currentView === item.id }"
          @click="currentView = item.id"
          :title="item.label"
        >
          <component :is="item.icon" :size="19" aria-hidden="true" />
          <span>{{ item.label }}</span>
        </button>
      </nav>

      <div class="sidebar-footer">
        <p>API</p>
        <strong>{{ apiStatusLabel }}</strong>
      </div>
    </aside>

    <main class="workspace">
      <header class="topbar">
        <div>
          <p class="eyebrow">Used Car Desk</p>
          <h1>{{ pageTitle }}</h1>
        </div>
        <div class="topbar-actions">
          <button type="button" class="icon-button" title="再読み込み" @click="loadVehicles">
            <RefreshCw :size="19" aria-hidden="true" />
          </button>
          <button type="button" class="icon-button" title="CSVを書き出す" @click="exportVehicles">
            <Download :size="19" aria-hidden="true" />
          </button>
          <button type="button" class="primary-button" @click="openCreateModal">
            <Plus :size="18" aria-hidden="true" />
            <span>車両を追加</span>
          </button>
        </div>
      </header>

      <div v-if="apiError" class="api-banner">
        {{ apiError }}
      </div>

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
              <h2>在庫車両一覧</h2>
              <p>{{ isLoading ? '読み込み中...' : `全 ${filteredVehicles.length} 件` }}</p>
            </div>
            <div class="filters">
              <label class="search-field">
                <Search :size="18" aria-hidden="true" />
                <input v-model.trim="query" type="search" placeholder="車名、管理番号、店舗で検索" />
              </label>
              <select v-model="makerFilter" aria-label="メーカーで絞り込み">
                <option value="all">すべてのメーカー</option>
                <option v-for="maker in makers" :key="maker" :value="maker">{{ maker }}</option>
              </select>
              <select v-model="statusFilter" aria-label="状態で絞り込み">
                <option value="all">すべての状態</option>
                <option v-for="status in statusOptions" :key="status" :value="status">{{ statusLabels[status] }}</option>
              </select>
            </div>
          </div>

          <div class="table-wrap">
            <table class="vehicle-table">
              <thead>
                <tr>
                  <th>車両</th>
                  <th>管理番号</th>
                  <th>メーカー</th>
                  <th>年式</th>
                  <th>走行距離</th>
                  <th>価格</th>
                  <th>状態</th>
                  <th>店舗</th>
                  <th class="actions-col">操作</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="vehicle in filteredVehicles" :key="vehicle.id">
                  <td>
                    <div class="vehicle-cell">
                      <div class="avatar">{{ vehicle.maker.slice(0, 1) }}</div>
                      <div>
                        <strong>{{ vehicle.name }}</strong>
                        <span>{{ vehicle.fuel }} · {{ vehicle.transmission }} · 車検 {{ vehicle.inspection }}</span>
                      </div>
                    </div>
                  </td>
                  <td>{{ vehicle.stockNo }}</td>
                  <td>{{ vehicle.maker }}</td>
                  <td>{{ vehicle.year }}年</td>
                  <td>{{ formatMileage(vehicle.mileage) }}</td>
                  <td>{{ formatPrice(vehicle.price) }}</td>
                  <td><span :class="['status-pill', statusTone(vehicle.status)]">{{ statusLabels[vehicle.status] }}</span></td>
                  <td>{{ vehicle.store }}</td>
                  <td>
                    <div class="row-actions">
                      <button type="button" title="編集" @click="openEditModal(vehicle)">
                        <Pencil :size="17" aria-hidden="true" />
                      </button>
                      <button type="button" title="削除" @click="removeVehicle(vehicle.id)">
                        <Trash2 :size="17" aria-hidden="true" />
                      </button>
                    </div>
                  </td>
                </tr>
                <tr v-if="filteredVehicles.length === 0">
                  <td colspan="9" class="empty-state">条件に一致する車両が見つかりません</td>
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
              <h2>価格帯分布</h2>
              <p>現在の在庫価格をもとに集計</p>
            </div>
          </div>
          <div class="bars">
            <div v-for="bucket in priceBuckets" :key="bucket.label" class="bar-row">
              <span>{{ bucket.label }}</span>
              <div class="bar-track"><i :style="{ width: bucket.percent + '%' }"></i></div>
              <strong>{{ bucket.count }}</strong>
            </div>
          </div>
        </article>

        <article class="panel ranking-panel">
          <div class="panel-header">
            <div>
              <h2>低走行おすすめ</h2>
              <p>走行距離が短い車両を優先表示</p>
            </div>
          </div>
          <div class="ranking-list">
            <div v-for="(vehicle, index) in lowMileageVehicles" :key="vehicle.id" class="ranking-item">
              <span>{{ index + 1 }}</span>
              <div>
                <strong>{{ vehicle.name }}</strong>
                <p>{{ vehicle.year }}年 · {{ vehicle.maker }}</p>
              </div>
              <em>{{ formatMileage(vehicle.mileage) }}</em>
            </div>
          </div>
        </article>
      </section>

      <section v-else class="content-grid">
        <article class="panel">
          <div class="panel-header">
            <div>
              <h2>メーカー別概覧</h2>
              <p>在庫台数と平均価格</p>
            </div>
          </div>
          <div class="class-list">
            <div v-for="item in makerSummary" :key="item.maker" class="class-item">
              <div>
                <strong>{{ item.maker }}</strong>
                <span>{{ item.count }} 台</span>
              </div>
              <p>{{ formatPrice(item.averagePrice) }}</p>
            </div>
          </div>
        </article>

        <article class="panel notice-panel">
          <div class="panel-header">
            <div>
              <h2>対応リマインド</h2>
              <p>状態、走行距離、車検データから作成</p>
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
      <form class="vehicle-modal" @submit.prevent="saveVehicle">
        <div class="modal-header">
          <div>
            <p class="eyebrow">{{ editingId ? 'Edit Vehicle' : 'New Vehicle' }}</p>
            <h2>{{ editingId ? '車両を編集' : '車両を追加' }}</h2>
          </div>
          <button type="button" class="icon-button" title="閉じる" @click="closeModal">
            <X :size="19" aria-hidden="true" />
          </button>
        </div>

        <div class="form-grid">
          <label>
            車名
            <input v-model.trim="form.name" required />
          </label>
          <label>
            管理番号
            <input v-model.trim="form.stockNo" required />
          </label>
          <label>
            メーカー
            <select v-model="form.maker" required>
              <option v-for="maker in makers" :key="maker" :value="maker">{{ maker }}</option>
            </select>
          </label>
          <label>
            年式
            <input v-model.number="form.year" type="number" min="1990" max="2030" required />
          </label>
          <label>
            走行距離
            <input v-model.number="form.mileage" type="number" min="0" required />
          </label>
          <label>
            価格
            <input v-model.number="form.price" type="number" min="0" step="10000" required />
          </label>
          <label>
            燃料
            <input v-model.trim="form.fuel" required />
          </label>
          <label>
            ミッション
            <input v-model.trim="form.transmission" required />
          </label>
          <label>
            車検
            <input v-model.trim="form.inspection" placeholder="2027/05" required />
          </label>
          <label>
            店舗
            <select v-model="form.store" required>
              <option v-for="store in stores" :key="store" :value="store">{{ store }}</option>
            </select>
          </label>
          <label class="full">
            状態
            <select v-model="form.status">
              <option v-for="status in statusOptions" :key="status" :value="status">{{ statusLabels[status] }}</option>
            </select>
          </label>
        </div>

        <div class="modal-actions">
          <button type="button" class="ghost-button" @click="closeModal">キャンセル</button>
          <button type="submit" class="primary-button">
            <Save :size="18" aria-hidden="true" />
            <span>保存</span>
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, reactive, ref } from 'vue'
import {
  BarChart3,
  CalendarClock,
  Car,
  Download,
  Gauge,
  LayoutDashboard,
  Pencil,
  Plus,
  RefreshCw,
  Save,
  Search,
  Tags,
  Trash2,
  TriangleAlert,
  Wrench,
  X
} from '@lucide/vue'

const apiUrl = '/api/vehicles'
const masterDataUrl = '/api/master-data'
const fallbackStatusOptions = ['available', 'reserved', 'sold', 'maintenance']

const fallbackStatusLabels = {
  available: '販売中',
  reserved: '商談中',
  sold: '成約済み',
  maintenance: '整備中'
}

const seedVehicles = [
  { id: 1, name: 'トヨタ プリウス Sツーリング', stockNo: 'UC-2026-001', maker: 'トヨタ', year: 2021, mileage: 24800, price: 2180000, status: 'available', store: '東京本店', fuel: 'ハイブリッド', transmission: 'AT', inspection: '2027/04' },
  { id: 2, name: 'ホンダ フィット e:HEV', stockNo: 'UC-2026-002', maker: 'ホンダ', year: 2022, mileage: 18200, price: 1680000, status: 'available', store: '横浜店', fuel: 'ハイブリッド', transmission: 'AT', inspection: '2027/09' },
  { id: 3, name: '日産 ノート X', stockNo: 'UC-2026-003', maker: '日産', year: 2020, mileage: 35600, price: 1320000, status: 'reserved', store: '千葉店', fuel: 'ガソリン', transmission: 'AT', inspection: '2026/11' },
  { id: 4, name: 'マツダ CX-5 XD', stockNo: 'UC-2026-004', maker: 'マツダ', year: 2019, mileage: 48600, price: 2360000, status: 'available', store: '東京本店', fuel: 'ディーゼル', transmission: 'AT', inspection: '2026/10' },
  { id: 5, name: 'スバル フォレスター Advance', stockNo: 'UC-2026-005', maker: 'スバル', year: 2021, mileage: 41200, price: 2740000, status: 'maintenance', store: '大宮店', fuel: 'ハイブリッド', transmission: 'AT', inspection: '2028/01' },
  { id: 6, name: 'レクサス UX250h', stockNo: 'UC-2026-006', maker: 'レクサス', year: 2022, mileage: 12600, price: 3980000, status: 'available', store: '横浜店', fuel: 'ハイブリッド', transmission: 'AT', inspection: '2027/12' },
  { id: 7, name: 'スズキ ジムニー XC', stockNo: 'UC-2026-007', maker: 'スズキ', year: 2020, mileage: 29800, price: 2260000, status: 'sold', store: '千葉店', fuel: 'ガソリン', transmission: 'MT', inspection: '2026/08' },
  { id: 8, name: 'BMW 320i M Sport', stockNo: 'UC-2026-008', maker: 'BMW', year: 2021, mileage: 33500, price: 3480000, status: 'available', store: '東京本店', fuel: 'ガソリン', transmission: 'AT', inspection: '2027/06' }
]

const vehicles = ref([])
const currentView = ref('overview')
const query = ref('')
const makerFilter = ref('all')
const statusFilter = ref('all')
const isModalOpen = ref(false)
const editingId = ref(null)
const isLoading = ref(false)
const apiError = ref('')
const apiConnected = ref(false)
const masterMakers = ref([])
const masterStores = ref([])
const masterStatuses = ref([])

const emptyForm = () => ({
  name: '',
  stockNo: '',
  maker: 'トヨタ',
  year: 2022,
  mileage: 10000,
  price: 1500000,
  status: 'available',
  store: '東京本店',
  fuel: 'ガソリン',
  transmission: 'AT',
  inspection: '2027/05'
})

const form = reactive(emptyForm())

document.documentElement.lang = 'ja'
document.title = '中古車管理システム'

const navItems = [
  { id: 'overview', label: '車両管理', icon: LayoutDashboard },
  { id: 'analytics', label: '販売分析', icon: BarChart3 },
  { id: 'makers', label: 'メーカー情報', icon: Tags }
]

const apiStatusLabel = computed(() => apiConnected.value ? 'Spring API 接続中' : 'デモデータ表示')
const pageTitle = computed(() => navItems.find(item => item.id === currentView.value)?.label || '車両管理')
const makers = computed(() => masterMakers.value.length ? masterMakers.value.map(maker => maker.name) : [...new Set(vehicles.value.map(vehicle => vehicle.maker))].sort())
const stores = computed(() => masterStores.value.length ? masterStores.value.map(store => store.name) : [...new Set(vehicles.value.map(vehicle => vehicle.store))].sort())
const statusOptions = computed(() => masterStatuses.value.length ? masterStatuses.value.map(status => status.code) : fallbackStatusOptions)
const statusLabels = computed(() => {
  if (!masterStatuses.value.length) return fallbackStatusLabels
  return masterStatuses.value.reduce((labels, status) => {
    labels[status.code] = status.label
    return labels
  }, {})
})

const filteredVehicles = computed(() => {
  const keyword = query.value.toLowerCase()
  return vehicles.value.filter(vehicle => {
    const matchQuery = [vehicle.name, vehicle.stockNo, vehicle.store, vehicle.maker].some(value => String(value).toLowerCase().includes(keyword))
    const matchMaker = makerFilter.value === 'all' || vehicle.maker === makerFilter.value
    const matchStatus = statusFilter.value === 'all' || vehicle.status === statusFilter.value
    return matchQuery && matchMaker && matchStatus
  })
})

const availableCount = computed(() => vehicles.value.filter(vehicle => vehicle.status === 'available').length)
const attentionCount = computed(() => vehicles.value.filter(vehicle => vehicle.status === 'maintenance' || vehicle.mileage >= 45000 || isInspectionSoon(vehicle.inspection)).length)
const averagePrice = computed(() => {
  if (!vehicles.value.length) return 0
  return Math.round(vehicles.value.reduce((sum, vehicle) => sum + Number(vehicle.price), 0) / vehicles.value.length)
})

const stats = computed(() => [
  { label: '在庫総数', value: vehicles.value.length, note: `${makers.value.length} メーカー`, icon: Car, tone: 'blue' },
  { label: '販売中', value: availableCount.value, note: '掲載可能な車両', icon: Gauge, tone: 'green' },
  { label: '平均価格', value: formatPrice(averagePrice.value), note: '現在の在庫から算出', icon: CalendarClock, tone: 'amber' },
  { label: '要確認', value: attentionCount.value, note: '整備・車検・走行距離', icon: TriangleAlert, tone: 'red' }
])

const lowMileageVehicles = computed(() => [...vehicles.value].sort((a, b) => a.mileage - b.mileage).slice(0, 5))

const priceBuckets = computed(() => {
  const buckets = [
    { label: '400万円+', min: 4000000, max: Number.POSITIVE_INFINITY },
    { label: '300-399', min: 3000000, max: 3999999 },
    { label: '200-299', min: 2000000, max: 2999999 },
    { label: '100-199', min: 1000000, max: 1999999 },
    { label: '0-99', min: 0, max: 999999 }
  ]
  const total = vehicles.value.length || 1
  return buckets.map(bucket => {
    const count = vehicles.value.filter(vehicle => vehicle.price >= bucket.min && vehicle.price <= bucket.max).length
    return { ...bucket, count, percent: Math.round((count / total) * 100) }
  })
})

const makerSummary = computed(() => makers.value.map(maker => {
  const members = vehicles.value.filter(vehicle => vehicle.maker === maker)
  const averagePrice = members.length
    ? Math.round(members.reduce((sum, vehicle) => sum + Number(vehicle.price), 0) / members.length)
    : 0
  return { maker, count: members.length, averagePrice }
}))

const todos = computed(() => [
  {
    title: `${vehicles.value.filter(vehicle => isInspectionSoon(vehicle.inspection)).length} 台の車検が近づいています`,
    desc: '車検期限を確認し、販売説明に反映してください',
    icon: CalendarClock
  },
  {
    title: `${vehicles.value.filter(vehicle => vehicle.status === 'maintenance').length} 台が整備中です`,
    desc: '整備完了後に販売ステータスを更新できます',
    icon: Wrench
  },
  {
    title: `${vehicles.value.filter(vehicle => vehicle.mileage >= 45000).length} 台が高走行です`,
    desc: '保証条件や点検履歴の確認をおすすめします',
    icon: TriangleAlert
  }
])

const requestJson = async (url, options = {}) => {
  const response = await fetch(url, {
    headers: { 'Content-Type': 'application/json' },
    ...options
  })
  if (!response.ok) {
    throw new Error(`API error: ${response.status}`)
  }
  if (response.status === 204) return null
  return response.json()
}

const loadVehicles = async () => {
  isLoading.value = true
  try {
    vehicles.value = await requestJson(apiUrl)
    apiConnected.value = true
    apiError.value = ''
  } catch {
    vehicles.value = seedVehicles
    apiConnected.value = false
    apiError.value = 'Spring Boot API に接続できないため、デモデータを表示しています。backend を起動すると DB データに切り替わります。'
  } finally {
    isLoading.value = false
  }
}

const loadMasterData = async () => {
  try {
    const masterData = await requestJson(masterDataUrl)
    masterMakers.value = masterData.makers || []
    masterStores.value = masterData.stores || []
    masterStatuses.value = masterData.statuses || []
  } catch {
    masterMakers.value = []
    masterStores.value = []
    masterStatuses.value = []
  }
}

const resetForm = values => {
  Object.assign(form, emptyForm(), values)
}

const openCreateModal = () => {
  editingId.value = null
  resetForm()
  if (makers.value[0]) form.maker = makers.value[0]
  if (stores.value[0]) form.store = stores.value[0]
  isModalOpen.value = true
}

const openEditModal = vehicle => {
  editingId.value = vehicle.id
  resetForm(vehicle)
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
}

const saveVehicle = async () => {
  const payload = {
    ...form,
    year: Number(form.year),
    mileage: Math.max(0, Number(form.mileage)),
    price: Math.max(0, Number(form.price))
  }

  if (apiConnected.value) {
    if (editingId.value) {
      await requestJson(`${apiUrl}/${editingId.value}`, {
        method: 'PUT',
        body: JSON.stringify(payload)
      })
    } else {
      await requestJson(apiUrl, {
        method: 'POST',
        body: JSON.stringify(payload)
      })
    }
    await loadVehicles()
  } else if (editingId.value) {
    vehicles.value = vehicles.value.map(vehicle => vehicle.id === editingId.value ? { ...vehicle, ...payload } : vehicle)
  } else {
    vehicles.value = [{ id: Date.now(), ...payload }, ...vehicles.value]
  }

  closeModal()
}

const removeVehicle = async id => {
  const target = vehicles.value.find(vehicle => vehicle.id === id)
  if (!target || !window.confirm(`${target.name} の記録を削除しますか？`)) return

  if (apiConnected.value) {
    await requestJson(`${apiUrl}/${id}`, { method: 'DELETE' })
    await loadVehicles()
  } else {
    vehicles.value = vehicles.value.filter(vehicle => vehicle.id !== id)
  }
}

const statusTone = status => {
  if (status === 'available') return 'ok'
  if (status === 'sold') return 'pending'
  return 'danger'
}

const formatPrice = value => new Intl.NumberFormat('ja-JP', {
  style: 'currency',
  currency: 'JPY',
  maximumFractionDigits: 0
}).format(Number(value))

const formatMileage = value => `${new Intl.NumberFormat('ja-JP').format(Number(value))} km`

const isInspectionSoon = value => {
  const [year, month] = String(value).split('/').map(Number)
  if (!year || !month) return false
  const inspectionDate = new Date(year, month - 1, 1)
  const threshold = new Date()
  threshold.setMonth(threshold.getMonth() + 6)
  return inspectionDate <= threshold
}

const exportVehicles = () => {
  const header = ['車名', '管理番号', 'メーカー', '年式', '走行距離', '価格', '状態', '店舗', '燃料', 'ミッション', '車検']
  const rows = vehicles.value.map(vehicle => [
    vehicle.name,
    vehicle.stockNo,
    vehicle.maker,
    vehicle.year,
    vehicle.mileage,
    vehicle.price,
    statusLabels.value[vehicle.status],
    vehicle.store,
    vehicle.fuel,
    vehicle.transmission,
    vehicle.inspection
  ])
  const csv = [header, ...rows].map(row => row.map(cell => `"${String(cell).replaceAll('"', '""')}"`).join(',')).join('\n')
  const blob = new Blob([`\uFEFF${csv}`], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = 'used-cars-ja.csv'
  link.click()
  URL.revokeObjectURL(link.href)
}

onMounted(async () => {
  await Promise.all([loadMasterData(), loadVehicles()])
})
</script>
